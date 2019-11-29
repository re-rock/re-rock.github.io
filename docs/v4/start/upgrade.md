---
title: Upgrade Guide
---

バージョン3からバージョン4にアップグレードする場合、以下で述べる重要な変更点に注意する必要があります。

## PHP Version Requirement
Slim4はPHP7.1以上である必要があります。

## Breaking changes to Slim\App constructor
Slimのアプリ設定は、以前はコンテナーの一部でしたが、Slim4ではコンテナーから切り離されています。

```php
/**
 * Slim 3 App::__construct($container = [])
 * このように、設定はネストされていました
 */
$app = new App([
    'settings' => [...],
]);

/**
 * Slim 4 App::__constructor()メソッドは1つの必須パラメーターと4つのオプションパラメーターを取ります
 * 
 * @param ResponseFactoryInterface Any implementation of a ResponseFactory
 * @param ContainerInterface|null Any implementation of a Container
 * @param CallableResolverInterface|null Any implementation of a CallableResolver
 * @param RouteCollectorInterface|null Any implementation of a RouteCollector
 * @param RouteResolverInterface|null Any implementation of a RouteResolver
 */
$app = new App(...);
```

## Removed App Settings
- `addContentLengthHeader` この設定の新しい実装については、[Content Length Middleware](/docs/v4/middleware/content-length.html)を参照してください。
- `determineRouteBeforeAppMiddleware` ミドルウェアスタックの適切な位置に[Routing Middleware](/docs/v4/middleware/routing.html)を配置して、既存の動作を再現します。
- `outputBuffering` この設定の新しい実装については、[Output Buffering Middleware](/docs/v4/middleware/output-buffering.html)を参照してください。
- `displayErrorDetails` この設定の新しい実装については、[Error Handling Middleware](/docs/v4/middleware/error-handling.html)参照してください。

## Changes to Container
Slimはもはやコンテナーを持たないので、あなた自身で用意する必要があります。コンテナー内にあるリクエストまたはレスポンスに依存している場合は、自分でコンテナーに設定するか、リファクタリングする必要があります。
また、`App::__call()`メソッドが削除されたため、`$app->key_name()`を介してコンテナプロパティにアクセスすることはできなくなりました。

## Changes to Routing components
Slim3の`Router`コンポーネントは、FastRouteを`App`コアから切り離し、エンドユーザーにより高い柔軟性を提供するために、複数の異なるコンポーネントに分割されました。
具体的には`RouteCollector`、`RouteParser`、`RouteResolver`に分割されています。これらの3つのコンポーネントはすべて、独自に実装して`App`コンストラクターに挿入するための独自のインターフェイスをそれぞれ持ちます。
以下のプルリクエストは、これらの新しいコンポーネントのパブリックインターフェイスに関する多くの知見を提供します。
- [Pull Request #2604](https://github.com/slimphp/Slim/pull/2604)
- [Pull Request #2622](https://github.com/slimphp/Slim/pull/2622)
- [Pull Request #2639](https://github.com/slimphp/Slim/pull/2639)
- [Pull Request #2640](https://github.com/slimphp/Slim/pull/2640)
- [Pull Request #2641](https://github.com/slimphp/Slim/pull/2641)
- [Pull Request #2642](https://github.com/slimphp/Slim/pull/2642)

## New Middleware Approach
Slim4では、Slimアプリのコア機能やミドルウェア実装から分離することで、開発者により柔軟性を与えたいと考えていました。これにより、コアコンポーネントをカスタム実装へと代替できます。

## Middleware Execution
ミドルウェア実行は変更されておらず、Slim3のように引き続き `Last In First Out (LIFO)`です。

## New App Factory
`AppFactory`コンポーネントは、PSR-7実装を`App`から切り離すことによって生じる摩擦を軽減するために導入されました。
プロジェクトルートにインストールされているPSR-7実装およびServerRequestクリエーターを検出し、`AppFactory::create()`を介してアプリをインスタンス化し、`ServerRequest`オブジェクトを渡さなくても`App::run()`を使用できるようにします。
次のPSR-7実装と`ServerRequest`クリエーターコンボがサポートされています。
- [Slim PSR-7](https://github.com/slimphp/Slim-Psr7)
- [Nyholm PSR-7](https://github.com/Nyholm/psr7) and [Nyholm PSR-7 Server](https://github.com/Nyholm/psr7-server)
- [Guzzle PSR-7](https://github.com/guzzle/psr7) and [Guzzle HTTP Factory](https://github.com/http-interop/http-factory-guzzle)
- [Zend Diactoros](https://github.com/zendframework/zend-diactoros)

## New Routing Middleware

ルーティングはミドルウェアとして実装されています。引き続きルーティングに必要な[FastRoute](https://github.com/nikic/FastRoute)を使用します。
もし`determineRouteBeforeAppMiddleware`を使用する場合、`run()`を呼び出す前に以前の動作を維持するため、アプリケーションに`Middleware\RoutingMiddleware`ミドルウェアを追加する必要があります。
詳細は[Pull Request #2288](https://github.com/slimphp/Slim/pull/2288)を参照してください。

```php
<?php
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

// Add Routing Middleware
$app->addRoutingMiddleware();

// ...

$app->run();
```

## New Error Handling Middleware
エラー処理もミドルウェアとして実装されています。
詳細については、[Pull Request #2398](https://github.com/slimphp/Slim/pull/2398)参照してください。

```php
<?php
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

/*
 * ルーティングミドルウェアは、ErrorMiddlewareの前に追加する必要があります 
 * 追加しないと、そこからスローされた例外は処理されません
 */
$app->addRoutingMiddleware();

/*
 * Add Error Handling Middleware
 *
 * @param bool $displayErrorDetails -> 本番環境ではfalseに設定する必要があります
 * @param bool $logErrors -> パラメータはデフォルトのErrorHandlerに渡されます
 * @param bool $logErrorDetails -> エラーログにエラーの詳細を表示します
 * これはあなたが選択したCallableオブジェクトに置き換えることができます。
 
 * 注）このミドルウェアは1番最後に追加する必要があります。
 * この後にミドルウェア用に追加されたとしても、例外/エラー処理は行われません。
 */
$app->addErrorMiddleware(true, true, true);

// ...

$app->run();
```

## New Dispatcher & Routing Results

リザルトラッパーの追加と、例外発生時だけでなくルートで許可されたメソッドの一覧へのアクセスができるFastRouteディスパッチャーのラッパーを作成しました。
リクエスト属性の`routeInfo`は廃止され、`routingResults`に置き換えられました。
詳細については[Pull Request #2405](https://github.com/slimphp/Slim/pull/2405)を参照してください。

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;
use Slim\Routing\RouteContext;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/hello/{name}', function (Request $request, Response $response) {
    $routeContext = RouteContext::fromRequest($request);
    $routingResults = $routeContext->getRoutingResults();
    
    // ルートの解析された引数をすべて取得します。例）['name' => 'John']
    $routeArguments = $routingResults->getRouteArguments();
    // Slim3のようにエラーが発生したときだけでなく、ルートで許可されているメソッドが常に利用可能です
    $allowedMethods = $routingResults->getAllowedMethods();
    
    return $response;
});

// ...

$app->run();
```

## New Method Overriding Middleware
カスタムヘッダーまたはボディパラメーターを使用してHTTPメソッドをオーバーライドする場合、以前のようにメソッドをオーバーライドできるように、`Middleware\MethodOverrideMiddleware`ミドルウェアを追加する必要があります。
詳細については、[Pull Request #2329](https://github.com/slimphp/Slim/pull/2329)を参照してください。

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Middleware\MethodOverridingMiddleware;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$methodOverridingMiddleware = new MethodOverridingMiddleware();
$app->add($methodOverridingMiddleware);

// ...

$app->run();
```


## New Content Length Middleware
コンテンツ長ミドルウェアは、レスポンスに`Content-Length`ヘッダーを自動的に追加します。これは、Slim3で削除された`addContentLengthHeader`設定を置き換えるためです。
このミドルウェアは、最後に実行されるようにミドルウェアスタックの中央に配置する必要があります。

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Middleware\ContentLengthMiddleware;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$contentLengthMiddleware = new ContentLengthMiddleware();
$app->add($contentLengthMiddleware);

// ...

$app->run();
```

## New Output Buffering Middleware
出力バッファリングミドルウェアを使用すると、出力バッファリングの2つのモードの`APPEND`（デフォルト）と`PREPEND`を切り替えることができます。
`APPEND`モードは既存のレスポンスボディを使用してコンテンツを追加し、`PREPEND`モードは新しいレスポンスボディを作成して既存のレスポンスに追加します。
このミドルウェアは、最後に実行されるように、ミドルウェアスタックの中央に配置する必要があります。

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Middleware\OutputBufferingMiddleware;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

/**
 * 2つのモードが利用可能です
 * OutputBufferingMiddleware::APPEND (default mode) - 既存のレスポンスボディに追加します
 * OutputBufferingMiddleware::PREPEND - 完全に新しいレスポンスボディを作成します
 */
$mode = OutputBufferingMiddleware::APPEND;
$outputBufferingMiddleware = new OutputBufferingMiddleware($mode);

// ...

$app->run();
```
