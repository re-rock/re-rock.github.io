---
title: Dependency Container
---

Slimは依存関係コンテナを使用して、アプリケーションの依存関係を準備、管理、および注入を行います。
Slimは、[PSR-11](http://www.php-fig.org/psr/psr-11/)または[Container-Interop](https://github.com/container-interop/container-interop)インターフェイスを実装するコンテナをサポートします。
Slimの組み込みコンテナ（[Pimple](http://pimple.sensiolabs.org/)をベース）または[Acclimate](https://github.com/jeremeamia/acclimate-container)や[PHP-DI](http://php-di.org/doc/frameworks/slim.htmlなどのサードパーティコンテナーを使用できます。

## How to use the container

依存関係コンテナを自分で用意する必要はありません。もし自分で用意した場合は、コンテナインスタンスをSlimアプリケーションのコンストラクタに注入する必要があります。

```php
$container = new \Slim\Container;
$app = new \Slim\App($container);
```

Slimコンテナにサービスを追加する

```php
$container = $app->getContainer();
$container['myService'] = function ($container) {
    $myService = new MyService();
    return $myService;
};
```

コンテナから明示的または暗黙的にサービスを取得できます。次のように、Slimアプリケーションルート内からコンテナインスタンスへの明示的な参照を取得できます。

```php
/**
 * Example GET route
 *
 * @param  \Psr\Http\Message\ServerRequestInterface $req  PSR7 request
 * @param  \Psr\Http\Message\ResponseInterface      $res  PSR7 response
 * @param  array                                    $args Route parameters
 *
 * @return \Psr\Http\Message\ResponseInterface
 */
$app->get('/foo', function ($req, $res, $args) {
    $myService = $this->get('myService');

    return $res;
});
```

下記は、コンテナから暗黙的にサービスを取得しています。

```php
/**
 * Example GET route
 *
 * @param  \Psr\Http\Message\ServerRequestInterface $req  PSR7 request
 * @param  \Psr\Http\Message\ResponseInterface      $res  PSR7 response
 * @param  array                                    $args Route parameters
 *
 * @return \Psr\Http\Message\ResponseInterface
 */
$app->get('/foo', function ($req, $res, $args) {
    $myService = $this->myService;

    return $res;
});
```

使用する前にコンテナにサービスが存在するかどうかをテストするには、次のように`has()`メソッドを使用します。

```php
/**
 * Example GET route
 *
 * @param  \Psr\Http\Message\ServerRequestInterface $req  PSR7 request
 * @param  \Psr\Http\Message\ResponseInterface      $res  PSR7 response
 * @param  array                                    $args Route parameters
 *
 * @return \Psr\Http\Message\ResponseInterface
 */
$app->get('/foo', function ($req, $res, $args) {
    if($this->has('myService')) {
        $myService = $this->myService;
    }

    return $res;
});
```

Slimは`__get()` および `__isset()` マジックメソッドを使用します。これらのメソッドは、アプリケーションインスタンスにまだ存在しないすべてのプロパティに対して、アプリケーションコンテナに従います。

## Required services

コンテナはこれらの必要なサービスを実装する必要があります。 Slimの組み込みコンテナを使用する場合はこれらがデフォルトで提供されます。サードパーティのコンテナを選択する場合、これらの必要なサービスを独自に定義する必要があります。

settings
:   キーを含むアプリケーション設定の連想配列:
    
    * `httpVersion`
    * `responseChunkSize`
    * `outputBuffering`
    * `determineRouteBeforeAppMiddleware`.
    * `displayErrorDetails`.
    * `addContentLengthHeader`.
    * `routerCacheFile`.

環境
:   `\Slim\Http\Environment`インスタンス.

リクエスト
:   `\Psr\Http\Message\ServerRequestInterface`インスタンス.

レスポンス
:   `\Psr\Http\Message\ResponseInterface`インスタンス.

ルータ
:   `\Slim\Interfaces\RouterInterface`インスタンス.

ハンドラー
:   `\Slim\Interfaces\InvocationStrategyInterface`インスタンス.

phpErrorHandler
:   PHP7エラーがスローされた場合に呼び出されるCallable。 Callableは`\Psr\Http\Message\ResponseInterface`のインスタンスを返さなければならず、下記の3つの引数を受け入れます      :

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`
3. `\Error`

errorHandler
:   例外がスローされた場合に呼び出されるCallable。 Callableは `\Psr\Http\Message\ResponseInterface` のインスタンスを返さなければならず、下記の3つの引数を受け入れます :

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`
3. `\Exception`

notFoundHandler
:   現在のHTTPリクエストURIがアプリケーションルートと一致しない場合に呼び出されるCallable。呼び出し可能オブジェクトは、`\Psr\Http\Message\ResponseInterface`のインスタンスを返さなければならず、下記の2つの引数を受け入れなければなりません :

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`

notAllowedHandler
: アプリケーションルートが現在のHTTPリクエストパスには一致するが、メソッドが一致していない場合に呼び出されるCallable。 callableは`\Psr\Http\Message\ResponseInterface` のインスタンスを返さなければならず、下記の3つの引数を受け入れます：

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`
3. Array of allowed HTTP methods

callableResolver
:   `\Slim\Interfaces\CallableResolverInterface`のインスタンス.
