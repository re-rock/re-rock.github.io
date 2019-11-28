---
title: Action-Domain-Responder with Slim
---

ここでは、Slimチュートリアルアプリケーションをリファクタリングして、[Action-Domain-Responder](http://pmjones.io/adr)パターンにさらに忠実に従う方法を紹介します。

Slim（および他のほとんどの[HTTP user interface frameworks](http://paul-m-jones.com/archives/6627)）の良い点の1つは、それらが既に"action"指向であることです。
つまり、ルーターは多くのアクションメソッドを持つコントローラークラスを想定していません。代わりに、アクションクロージャまたはシングルアクション呼び出しクラスを想定しています。

そのため、SlimにはAction-Domain-ResponderのAction部分が既に存在します。必要なのは、アクションから無関係な処理を排除して、アクションの動作をドメインおよびレスポンダーの動作からより明確に分離することです。

## Extract Domain

ドメインロジックを抽出することから始めましょう。元のチュートリアルでは、アクションは2つのデータソースマッパーを直接使用し、いくつかのビジネスロジックも埋め込みます。
`TicketService`というサービス層クラスを作成し、それらの操作をアクションからドメインに移動できます。そうすると、このクラスを取得できます。

```php
class TicketService
{
    protected $ticket_mapper;
    protected $component_mapper;

    public function __construct(
        TicketMapper $ticket_mapper,
        ComponentMapper $component_mapper
    ) {
        $this->ticket_mapper = $ticket_mapper;
        $this->component_mapper = $component_mapper;
    }

    public function getTickets()
    {
        return $this->ticket_mapper->getTickets();
    }

    public function getComponents()
    {
        return $this->component_mapper->getComponents();
    }

    public function getTicketById($ticket_id)
    {
        $ticket_id = (int) $ticket_id;
        return $this->ticket_mapper->getTicketById($ticket_id);
    }

    public function createTicket($data)
    {
        $component_id = (int) $data['component'];
        $component = $this->component_mapper->getComponentById($component_id);

        $ticket_data = [];
        $ticket_data['title'] = filter_var($data['title'], FILTER_SANITIZE_STRING);
        $ticket_data['description'] = filter_var($data['description'], FILTER_SANITIZE_STRING);
        $ticket_data['component'] = $component->getName();

        $ticket = new TicketEntity($ticket_data);
        $this->ticket_mapper->save($ticket);
        return $ticket;
    }
}
```

`index.php`で次のようにコンテナオブジェクトを作成します。

```php
$container['ticket_service'] = function ($c) {
    return new TicketService(
        new TicketMapper($c['db']),
        new ComponentMapper($c['db'])
    );
};
```

こうするとアクションはドメインロジックを直接実行する代わりに`TicketService`を使用できます。

```php
$app->get('/tickets', function (Request $request, Response $response) {
    $this->logger->addInfo("Ticket list");
    $tickets = $this->ticket_service->getTickets();
    $response = $this->view->render(
        $response,
        "tickets.phtml",
        ["tickets" => $tickets, "router" => $this->router]
    );
    return $response;
});

$app->get('/ticket/new', function (Request $request, Response $response) {
    $components = $this->ticket_service->getComponents();
    $response = $this->view->render(
        $response,
        "ticketadd.phtml",
        ["components" => $components]
    );
    return $response;
});

$app->post('/ticket/new', function (Request $request, Response $response) {
    $data = $request->getParsedBody();
    $this->ticket_service->createTicket($data);
    $response = $response->withRedirect("/tickets");
    return $response;
});

$app->get('/ticket/{id}', function (Request $request, Response $response, $args) {
    $ticket = $this->ticket_service->getTicketById($args['id']);
    $response = $this->view->render(
        $response,
        "ticketdetail.phtml",
        ["ticket" => $ticket]
    );
    return $response;
})->setName('ticket-detail');
```

ここでの利点の1つは、アクションとは分離してドメインアクティビティをテストできることです。エンドツーエンドのシステムテストではなく、統合テスト、単体テストなどを開始できます。

## Extract Responder

In the case of the tutorial application, the presentation work is so straightforward as to not require a separate Responder for each action. A relaxed variation of a Responder layer is perfectly suitable in this simple case, one where each Action uses a different method on a common Responder.

Extracting the presentation work to a separate Responder, so that response-building is completely removed from the Action, looks like this:

```php
use Psr\Http\Message\ResponseInterface as Response;
use Slim\Views\PhpRenderer;

class TicketResponder
{
    protected $view;

    public function __construct(PhpRenderer $view)
    {
        $this->view = $view;
    }

    public function index(Response $response, array $data)
    {
        return $this->view->render(
            $response,
            "tickets.phtml",
            $data
        );
    }

    public function detail(Response $response, array $data)
    {
        return $this->view->render(
            $response,
            "ticketdetail.phtml",
            $data
        );
    }

    public function add(Response $response, array $data)
    {
        return $this->view->render(
            $response,
            "ticketadd.phtml",
            $data
        );
    }

    public function create(Response $response)
    {
        return $response->withRedirect("/tickets");
    }
}
```

We can then add the `TicketResponder` object to the container in `index.php`:

```php
$container['ticket_responder'] = function ($c) {
    return new TicketResponder($c['view']);
};
```

And finally we can refer to the Responder, instead of just the template system, in the Actions:

```php
$app->get('/tickets', function (Request $request, Response $response) {
    $this->logger->addInfo("Ticket list");
    $tickets = $this->ticket_service->getTickets();
    return $this->ticket_responder->index(
        $response,
        ["tickets" => $tickets, "router" => $this->router]
    );
});

$app->get('/ticket/new', function (Request $request, Response $response) {
    $components = $this->ticket_service->getComponents();
    return $this->ticket_responder->add(
        $response,
        ["components" => $components]
    );
});

$app->post('/ticket/new', function (Request $request, Response $response) {
    $data = $request->getParsedBody();
    $this->ticket_service->createTicket($data);
    return $this->ticket_responder->create($response);
});

$app->get('/ticket/{id}', function (Request $request, Response $response, $args) {
    $ticket = $this->ticket_service->getTicketById($args['id']);
    return $this->ticket_responder->detail(
        $response,
        ["ticket" => $ticket]
    );
})->setName('ticket-detail');
```

Now we can test the response-building work separately from the domain work.

Some notes:

Putting all the response-building in a single class with multiple methods, especially for simple cases like this tutorial, is fine to start with. For ADR, is not strictly necessary to have one Responder for each Action. What *is* necessary is to extract the response-building concerns out of the Action.

But as the presentation logic complexity increases (content-type negotiation? status headers? etc.), and as dependencies become different for each kind of response being built, you will want to have a Responder for each Action.

Alternatively, you might stick with a single Responder, but reduce its interface to a single method. In that case, you may find that using a [Domain Payload](http://paul-m-jones.com/archives/6043) (instead of "naked" domain results) has some significant benefits.

## Conclusion

At this point, the Slim tutorial application has been converted to ADR. We have separated the domain logic to a `TicketService`, and the presentation logic to a `TicketResponder`. And it's easy to see how each Action does pretty much the same thing:

- Marshals input and passes it into the Domain
- Gets back a result from the Domain and passes it to the Responder
- Invokes the Responder so it can build and return the Response

Now, for a simple case like this, using ADR (or even webbishy MVC) might seem like overkill. But simple cases become complex quickly, and this simple case shows how the ADR separation-of-concerns can be applied as a Slim-based application increases in complexity.
