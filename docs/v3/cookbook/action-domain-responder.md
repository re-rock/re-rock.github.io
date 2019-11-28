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

このチュートリアルアプリケーションの場合、プレゼンテーションの処理は非常にシンプルなので、各アクションごとに個別のレスポンダーは必要としません。
このようなシンプルなケースでは、各アクションが共通のレスポンダーで異なるメソッドを使用する、規制のゆるいレスポンダーレイヤーのバリエーションが最適です。

以下ように、プレゼンテーションの作業を別のレスポンダーに抽出し、レスポンスの構築がアクションから完全に排除されます。

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

その後、`index.php`で`TicketResponder`オブジェクトをコンテナーに追加できます。

```php
$container['ticket_responder'] = function ($c) {
    return new TicketResponder($c['view']);
};
```

最後には、テンプレートシステムの代わりにアクションでレススポンダーを参照できるようになります。

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

これで、ドメイン作業とは分離してレスポンス処理をテストできます。

メモ:

とくにこのチュートリアルのようなシンプルなケースでは、すべてのレスポンス処理を複数のメソッドを持つ単一のクラスに配置することから始めてください。
ADRの場合、各アクションごとにそれぞれレスポンダーを用意する必要はありません。必要なのは、アクションからレスポンス処理の問題を排除することです。

しかし、プレゼンテーションロジックの複雑さが増すと（コンテンツタイプネゴシエーション？ステータスヘッダー？など）、構築されたレスポンスの種類ごとに依存関係が異なるため、アクションごとにレスポンダーが必要になります。

またはレスポンダーは1つだけで、かつその単一のメソッドのインターフェイスを減らしたいとします。その場合は[Domain Payload](http://paul-m-jones.com/archives/6043)を通常のドメインリザルトに代わって使用するといくつかの大きな恩恵を受けられます。

## Conclusion

この時点で、SlimチュートリアルアプリケーションはADRに変換されました。ドメインロジックを`TicketService`に、プレゼンテーションロジックを`TicketResponder`分離しました。
そして、各アクションがどのように同等の処理を行うか簡単に確認できます。

- インプットを整理し、それをドメインに渡します
- ドメインから結果を取得し、それをレスポンダーに渡します
- レスポンダーを呼び出して、レスポンスを作成して返すことができます

さて、このような単純なケースで、ADR（またはwebbishy MVC）を使用するのはやり過ぎのように思えるかもしれません。
しかし、シンプルなケースはすぐに複雑になり、このケースはSlimベースのアプリケーションの複雑さが増すにつれて、ADRの"関心の分離"をどのように適用できるかを示しています。
