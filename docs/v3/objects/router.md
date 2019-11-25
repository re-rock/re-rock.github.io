---
title: Router
---

Slim Frameworkのルーターは[nikic/fastroute](https://github.com/nikic/FastRoute)コンポーネント上に構築されており、非常に高速で安定しています。

## How to create routes

`\Slim\App`インスタンスでプロキシメソッドを使用して、アプリケーションルートを定義できます。 Slim Frameworkは、一般的なHTTPメソッド用のメソッドを提供します。

### GET Route

Slimアプリケーションの`get()`メソッドを使用して、`GET`HTTPリクエストのみを処理するルートを追加できます。
次の2つの引数を受け入れます。

1. ルートパターン（オプションとしてプレースホルダー）
2. ルートコールバック

```php
$app = new \Slim\App();
$app->get('/books/{id}', function ($request, $response, $args) {
    // Show book identified by $args['id']
});
```

### POST Route

Slimアプリケーションの`post()`メソッドを使用して、`POST` HTTPリクエストのみを処理するルートを追加できます。
次の2つの引数を受け入れます。

1. ルートパターン（オプションとしてプレースホルダー）
2. ルートコールバック

```php
$app = new \Slim\App();
$app->post('/books', function ($request, $response, $args) {
    // Create new book
});
```

### PUT Route

Slimアプリケーションの`put()`メソッドを使用して、`PUT` HTTPリクエストのみを処理するルートを追加できます。
次の2つの引数を受け入れます。

1. ルートパターン（オプションとしてプレースホルダー）
2. ルートコールバック

```php
$app = new \Slim\App();
$app->put('/books/{id}', function ($request, $response, $args) {
    // Update book identified by $args['id']
});
```

### DELETE Route

Slimアプリケーションの`delete()`メソッドを使用して、`DELETE` HTTPリクエストのみを処理するルートを追加できます。
次の2つの引数を受け入れます。

1. ルートパターン（オプションとしてプレースホルダー）
2. ルートコールバック

```php
$app = new \Slim\App();
$app->delete('/books/{id}', function ($request, $response, $args) {
    // Delete book identified by $args['id']
});
```

### OPTIONS Route

Slimアプリケーションの`options()`メソッドを使用して、`OPTIONS` HTTPリクエストのみを処理するルートを追加できます。
次の2つの引数を受け入れます。

1. ルートパターン（オプションとしてプレースホルダー）
2. ルートコールバック

```php
$app = new \Slim\App();
$app->options('/books/{id}', function ($request, $response, $args) {
    // Return response headers
});
```

### PATCH Route

Slimアプリケーションの`patch()`メソッドを使用して、`PATCH` HTTPリクエストのみを処理するルートを追加できます。
次の2つの引数を受け入れます。

1. ルートパターン（オプションとしてプレースホルダー）
2. ルートコールバック

```php
$app = new \Slim\App();
$app->patch('/books/{id}', function ($request, $response, $args) {
    // Apply changes to book identified by $args['id']
});
```

### Any Route

Slimアプリケーションの`any()`メソッドを使用して、すべてのHTTPリクエストメソッドを処理するルートを追加できます。
次の2つの引数を受け入れます。

1. ルートパターン（オプションとしてプレースホルダー）
2. ルートコールバック

```php
$app = new \Slim\App();
$app->any('/books/[{id}]', function ($request, $response, $args) {
    // Apply changes to books or book identified by $args['id'] if specified.
    // To check which method is used: $request->getMethod();
});
```

2番目のパラメーターはコールバックであることに注意してください。 クロージャの代わりにクラス（`__invoke()`の実装が必要）を指定できます。
その後、別の場所でマッピングを行うことができます。

```php
$app->any('/user', 'MyRestfulController');
```

### Custom Route

Slimアプリケーションの`map()`メソッドを使用して、複数のHTTPリクエストメソッドを処理するルートを追加できます。
次の3つの引数を受け入れます。

1. HTTPメソッドの配列
2. ルートパターン（オプションとしてプレースホルダー）
3. ルートコールバック

```php
$app = new \Slim\App();
$app->map(['GET', 'POST'], '/books', function ($request, $response, $args) {
    // Create new book or list all books
});
```

## Route callbacks

上記の各ルーティングメソッドは、最終引数としてコールバックルーティンを受け入れます。この引数には任意のPHP Callableを指定でき、デフォルトでは3つの引数を受け入れます。

**Request**
最初の引数は、進行中のHTTPリクエストを表す`Psr\Http\Message\ServerRequestInterface` オブジェクトです。

**Response**
2番目の引数は、進行中のHTTPレスポンスを表す`Psr\Http\Message\ResponseInterface`オブジェクトです。

**Arguments**
3番目の引数は、進行中のルートの名前付きプレースホルダーの値を含む連想配列です。

### Writing content to the response

HTTPレスポンスにコンテンツを書き込む方法は2つあります。1つ目は、ルートコールバックからコンテンツを`echo()` するだけです。このコンテンツは、進行中のHTTPレスポンスオブジェクトに追加されます。
2つ目は、`Psr\Http\Message\ResponseInterface`オブジェクトを返すことができます。

### Closure binding

ルートコールバックとして `Closure`インスタンスを使用する場合、クロージャーの状態は`Container`インスタンスにバインドされます。
これは、`$this`キーワードを使用してクロージャー内のDIコンテナインスタンスにアクセスできることを意味します。

```php
$app = new \Slim\App();
$app->get('/hello/{name}', function ($request, $response, $args) {
    // Use app HTTP cookie service
    $this->get('cookies')->set('name', [
        'value' => $args['name'],
        'expires' => '7 days'
    ]);
});
```

## Redirect helper

Slimアプリケーションの`redirect()`メソッドを使用して、`GET`HTTPリクエストを別のURLにリダイレクトするルートを追加できます。次の3つの引数を受け入れます。

1. リダイレクト元のルートパターン（オプションとしてプレースホルダー）
2. リダイレクト先のパス。文字列または[`Psr\Http\Message\UriInterface`](https://www.php-fig.org/psr/psr-7/#35-psrhttpmessageuriinterface)
3. 使用するHTTPステータスコード（オプション。設定されていない場合は`302`）

```php
$app = new \Slim\App();
$app->redirect('/books', '/library', 301);
```

`redirect()`ルートは、リクエストされたステータスコードと2番目の引数に設定された`Location`ヘッダーに対して応答します。

## Route strategies

ルートコールバックのシグネチャは、ルートストラテジによって決定されます。デフォルトでは、Slimはルートコールバックがリクエスト、レスポンス、およびルートプレースホルダー引数の配列を受け入れることを想定しています。これはRequestResponseストラテジと呼ばれます。ただし、代替ストラテジを使用するだけで、期待されるルートコールバックシグネチャを変更できます。例として、SlimはRequestResponse引数と呼ばれる代替ストラテジを提供します。このストラテジは、リクエストとレスポンスに加えて、各ルートプレースホルダーを個別の引数として受け入れます。この代替ストラテジの使用例を次に示します。デフォルトの`\Slim\Container`によって提供される`foundHandler`DIに置き換えるだけです。

```php
$c = new \Slim\Container();
$c['foundHandler'] = function() {
    return new \Slim\Handlers\Strategies\RequestResponseArgs();
};

$app = new \Slim\App($c);
$app->get('/hello/{name}', function ($request, $response, $name) {
    return $response->write($name);
});
```
`\Slim\Interfaces\InvocationStrategyInterface`を実装することにより、独自のルートストラテジを提供できます。

## Route placeholders

上記で紹介した各ルーティング方法は、HTTPリクエストURIと照合されるURLパターンを受け入れます。ルートパターンは、名前付きプレースホルダーを使用して、HTTPリクエストURIセグメントを動的に一致させることができます。

### Format

ルートパターンのプレースホルダーは `{` で始まり、その後にプレースホルダー名が続き、`}` で終わります。この例では `name` という名前のプレースホルダーです。

```php
$app = new \Slim\App();
$app->get('/hello/{name}', function ($request, $response, $args) {
    echo "Hello, " . $args['name'];
});
```

### Optional segments

URLのセクションをオプションにするには、角かっこで囲みます。

```php
$app->get('/users[/{id}]', function ($request, $response, $args) {
    // OK : `/users` and `/users/123`
    // NG : `/users/`
});
```

ネストにより、複数のオプションパラメーターもサポートされます。

```php
$app->get('/news[/{year}[/{month}]]', function ($request, $response, $args) {
    // reponds to `/news`, `/news/2016` and `/news/2016/03`
});
```

「無制限」のオプションパラメーターもサポートされています。

```php
$app->get('/news[/{params:.*}]', function ($request, $response, $args) {
    $params = explode('/', $args['params']);

    // $params is an array of all the optional segments
});
```

この例では、`/news/2016/03/20`のURIは、3つの要素を含む$ `$params`配列になります： `['2016', '03', '20']`。


### Regular expression matching

デフォルトでは、プレースホルダーは `{}` 内に記述され、任意の値を受け入れます。またプレースホルダーは、特定の正規表現に一致するHTTPリクエストURIを要求することもできます。HTTPリクエストURIがプレースホルダーの正規表現と一致しない場合、ルートは呼び出されません。下記は1つ以上の数値を必要とする`id`という名前のプレースホルダーの例です。

```php
$app = new \Slim\App();
$app->get('/users/{id:[0-9]+}', function ($request, $response, $args) {
    // Find user identified by $args['id']
});
```

## Route names

アプリケーションルートには名前を割り当てることができます。これは、ルーターの `pathFor()` メソッドを使用し、特定のルートへのURLをプログラムで生成する場合に便利です。
上記の各ルーティングメソッドは `\Slim\Route` オブジェクトを返し、このオブジェクトは `setName()` メソッドが公開されます。

Each routing method described above returns a `\Slim\Route` object, and this object exposes a `setName()` method.

```php
$app = new \Slim\App();
$app->get('/hello/{name}', function ($request, $response, $args) {
    echo "Hello, " . $args['name'];
})->setName('hello');
```

アプリケーションルーターの `pathFor()` メソッドを使用して、この名前付きルートのURLを生成できます。

```php
echo $app->getContainer()->get('router')->pathFor('hello', [
    'name' => 'Josh'
]);
// Outputs "/hello/Josh"
```

ルーターの`pathFor()`メソッドは2つの引数を受け入れます。

1. ルート名
2. ルートパターンのプレースホルダー配列と、置換用の値

## Route groups

ルートをグループにまとめられるように、`\Slim\App`は `group()` メソッドも用意しています。各グループのルートパターンは、その中に含まれるルートまたはグループの先頭に追加され、グループパターンのプレースホルダー引数はすべて、ネストされたルートで最終的に使用可能になります。

```php
$app = new \Slim\App();
$app->group('/users/{id:[0-9]+}', function (App $app) {
    $app->map(['GET', 'DELETE', 'PATCH', 'PUT'], '', function ($request, $response, $args) {
        // Find, delete, patch or replace user identified by $args['id']
    })->setName('user');
    $app->get('/reset-password', function ($request, $response, $args) {
        // Route for /users/{id:[0-9]+}/reset-password
        // Reset the password for user identified by $args['id']
    })->setName('user-password-reset');
});
```

グループパターンを空にして、共通のパターンを共有しないルートを論理的にグループ化できます。

```php
$app->group('', function(App $app) {
    $app->get('/billing', function ($request, $response, $args) {
        // Route for /billing
    });
    $this->get('/invoice/{id:[0-9]+}', function ($request, $response, $args) {
        // Route for /invoice/{id:[0-9]+}
    });
})->add( new SharedMiddleware() );
```

グループクロージャ内で、`$app`の代わりに`$this`を使用できます。コンテナインスタンスを使用したルートコールバックバインドの場合と同様に、Slimはクロージャーをアプリケーションインスタンスにバインドします。

* グループクロージャ内で、`$this`は`Slim\App`のインスタンスにバインドされます
* ルートクロージャ内で、`$this`は`Slim\App`のインスタンスにバインドされます

## Route middleware

ミドルウェアを任意のルートまたはルートグループに接続することもできます。[もっと詳しく](/docs/v3/concepts/middleware.html)。

## Router caching

デフォルトのSlimの設定で、有効なファイル名を設定することにより、ルーターのキャッシュを有効にできます。[もっと詳しく](/docs/v3/objects/application.html#slim-default-settings)。

## Container Resolution

ルートメソッドを定義するだけでなく、Slimにはさらにルートアクションメソッドを定義するいくつかの方法があります。

メソッドに加えて、次のものも使用できます。

 - `container_key:method`
 - `Class:method`
 - 呼び出し可能なクラス
 - `container_key`

この機能は、SlimのCallableリゾルバークラスによって有効になります。文字列のエントリーをメソッドの呼び出しに変換します。
例：

```php
$app->get('/', '\HomeController:home');
```

あるいは、PHPの`::class` 演算子を利用することもできます。この演算子はIDEルックアップシステムでうまく機能し、同様の結果を生成します。

```php
$app->get('/', \HomeController::class . ':home');
```

上記のコードでは `/` ルートを定義し、 `HomeController` クラスの `home()` メソッドを実行するようSlimに指示しています。

最初にSlimはコンテナーの`HomeController`のエントリーを探します。見つかった場合はそのインスタンスを使用し、
それ以外の場合は、コンテナーを最初の引数としてコンストラクターを呼び出します。クラスのインスタンスが作成されると、
定義したストラテジを使用して指定されたメソッドを呼び出します。

### Registering a controller with the container

`home`アクションメソッドでコントローラーを作成します。コンストラクターは、必要なDIを受け入れる必要があります。
例：

```php
class HomeController
{
    protected $view;

    public function __construct(\Slim\Views\Twig $view) {
        $this->view = $view;
    }
    public function home($request, $response, $args) {
      // your code here
      // use $this->view to render the HTML
      return $response;
    }
}
```

DIを使用してコントローラーをインスタンス化するコンテナーにファクトリーを作成します。

```php
$container = $app->getContainer();
$container['HomeController'] = function($c) {
    $view = $c->get("view"); // retrieve the 'view' from the container
    return new HomeController($view);
};
```

これにより、コンテナーをDIに利用できるため、特定の依存性をコントローラーに注入できます。

### Allow Slim to instantiate the controller

あるいはクラスのコンテナーにエントリーがない場合、Slimはコンテナーのインスタンスをコンストラクターに渡します。 
1つのアクションのみを処理する呼び出しクラスの代わりに、多くのアクションを持つコントローラーを構築できます。

```php
use Psr\Container\ContainerInterface;

class HomeController
{
   protected $container;

   // constructor receives container instance
   public function __construct(ContainerInterface $container) {
       $this->container = $container;
   }

   public function home($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }

   public function contact($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }
}
```

下記のようなコントローラーメソッドを使用できます。

```php
$app->get('/', \HomeController::class . ':home');
$app->get('/contact', \HomeController::class . ':contact');
```

### Using an invokable class

ルートcallableでメソッドを指定する必要はなく、次のような呼び出しクラスに設定するだけです。

```php
use Psr\Container\ContainerInterface

class HomeAction
{
   protected $container;

   public function __construct(ContainerInterface $container) {
       $this->container = $container;
   }

   public function __invoke($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }
}
```

このクラスは次のように使用できます。

```php
$app->get('/', \HomeAction::class);
```

繰り返しますが、コントローラーと同様にコンテナーにクラス名を登録すると、ファクトリーを作成し必要な特定の依存関係のみをアクションクラスに注入できます。
