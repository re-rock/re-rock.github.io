---
title: Upgrade Guide
---

Slimのバージョンを2から3に更新する場合、重要な変更点に注意してください。

## 必要とするPHPバージョン
Slim3 : PHP5.5以上

## クラス名を\Slim\Slimから\Slim\Appに変更
Slim3は通常`app`という名前のアプリケーションオブジェクトに`\Slim\App`を使用します。

```php
$app = new \Slim\App();
```

## 新しいルータ機能の特徴

```php
$app->get('/', function (Request $req,  Response $res, $args = []) {
    return $res->withStatus(400)->write('Bad Request');
});
```

## リクエストおよびレスポンスオブジェクトは、アプリケーションオブジェクトを介してアクセスできなくなりました。

Slim3はリクエストおよびレスポンスオブジェクトを引数としてルート処理関数に渡します。
ルート関数の本体で直接アクセスできるようになったため、リクエストとレスポンスは
 `/Slim/App` ([アプリケーション](/docs/v3/objects/application.html)オブジェクト)インスタンスの
 プロパティではなくなりました。

## GET / POST変数の取得

```php
$app->get('/', function (Request $req,  Response $res, $args = []) {
    $myvar1 = $req->getParam('myvar'); //checks both _GET and _POST [NOT PSR-7 Compliant]
    $myvar2 = $req->getParsedBody()['myvar']; //checks _POST  [IS PSR-7 compliant]
    $myvar3 = $req->getQueryParams()['myvar']; //checks _GET [IS PSR-7 compliant]
});
```

## Hooks

Slim3ではフックはフレームワークの一部ではなくなりました。代わりに、Slim2のデフォルトフックに
関連する機能を`middleware`として再実装することを検討してください。
コード内の任意のポイント（ルート内など）にカスタムフックを適用する機能が必要な場合は、
[Symfony's EventDispatcher](http://symfony.com/doc/current/components/event_dispatcher/introduction.html) や
[Zend Framework's EventManager](https://zend-eventmanager.readthedocs.org/en/latest/)などのサードパーティパッケージを検討する必要があります。

## HTTPキャッシュの削除
Slim3はHTTPキャシュを[Slim\Http\Cache](https://github.com/slimphp/Slim-HttpCache)という独自の
モジュールで削除します。

## Stop/Haltの廃止
SlimコアはStop / Haltを削除しました。アプリケーションではwithStatus()メソッドとwithBody()メソッド
の使用に移行する必要があります。

## autoloaderの廃止
`Slim::registerAutoloader()` は廃止され, 完全にcomposerに移行しました。

## コンテナの変更
`$app->container->singleton(...)`は `$container = $app->getContainer(); $container['...'] = function () {};`
に変更しました。詳細はPimpleのドキュメントをご覧ください。(Pimple : PHPで使えるシンプルな実装のDIコンテナ)

## configureMode()の廃止
`$app->configureMode(...)` はSlim3で廃止されました。

## PrettyExceptionsの廃止
多くの人たちの間で問題を引き起こすため廃止されました。

## Route::setDefaultConditions(...) の削除
ルートパターン内でデフォルト条件の正規表現を保持できるルータに切り替えました。

## リダイレクトの変更
Slim2ではリダイレクトリクエストのトリガーにヘルパー関数の `$app->redirect();`を使用します。 
Slim3ではレスポンスクラスを利用して同じことを行えます。

例:

```php
$app->get('/', function ($req, $res, $args) {
  return $res->withStatus(302)->withHeader('Location', 'your-new-uri');
});
```
また、他の処理を行わずにルートをリダイレクトしたい場合は、ショートカットヘルパー関数 `$app->redirect()` を
ルート定義として使用できます。

```php
$app->redirect('/', 'your-new-uri');
```

## ミドルウェアの機能
ミドルウェアの機能がクラスから関数に変更されました。

新機能

```php
use Psr\Http\Message\RequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

$app->add(function (Request $req,  Response $res, callable $next) {
    // 処理前に何かしら行う
    $newResponse = $next($req, $res);
    // ルートがレンダリングされたあとに何かしら行う
    return $newResponse; // continue
});
```

引き続きクラスも使用できます。

```php
namespace My;

use Psr\Http\Message\RequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

class Middleware
{
    function __invoke(Request $req,  Response $res, callable $next) {
        // 処理前に何かしら行う
        $newResponse = $next($req, $res);
        // ルートがレンダリングされたあとに何かしら行う
        return $newResponse; // continue
    }
}


// Register
$app->add(new My\Middleware());
// or
$app->add(My\Middleware::class);

```


## ミドルウェアの実行
ミドルウェアは1番最後に取り込まれ、1番最初に実行されます。

## Flash Messages
Flash messagesはSlim3コアの一部ではなくなり、代わりに[Slim Flash](/docs/v3/features/flash.html) 
パッケージとなりました。

## Cookies
Slim3ではクッキーがコアから削除されました。PSR-7互換のCookieコンポーネントについては
[FIG Cookies](https://github.com/dflydev/dflydev-fig-cookies)を参照してください。

## Cryptoの廃止
Slim3コアではcrypto(暗号化)との依存関係を廃止しました。

## New Router
Slimは現在、より強力な新しいルータである[FastRoute](https://github.com/nikic/FastRoute)を利用しています！

これは、オプションのセグメントに使用される中括弧と角括弧内の名前付きパラメーターによって、ルートパターンが変更されることを意味します。

```php
// named parameter:
$app->get('/hello/{name}', /*...*/);

// optional segment:
$app->get('/news[/{year}]', /*...*/);
```

## Route Middleware
Slim3ではルートミドルウェアを追加するための構文がわずかに変更されました。

```php
$app->get(…)->add($mw2)->add($mw1);
```

## Getting the current route
Slim3ではルートはリクエストオブジェクトの属性になります。

```php
$request->getAttribute('route');
```
ミドルウェアで現在のルートを取得する場合、アプリケーション構成で`determineRouteBeforeAppMiddleware`の
値を`true`に設定する必要があります。そうしないとgetAttributeコールは`null`を返します。

## ルータのurlFor() がpathFor()に変更

`urlFor()`は`pathFor()`に名称が変更され、`router`オブジェクトとして利用できます。

```php
$app->get('/', function ($request, $response, $args) {
    $url = $this->router->pathFor('home');
    $response->write("<a href='$url'>Home</a>");
    return $response;
})->setName('home');
```

また、`pathFor()`はbase pathに対応しています。

## Container and DI ... Constructing
SlimはDIコンテナとしてPimpleを使用します。

```php

// index.php
$app = new Slim\App(
    new \Slim\Container(
        include '../config/container.config.php'
    )
);
// Slimは、コンテナで定義されたHomeクラスを取得し、そのindexメソッドを実行します。 
// クラスがコンテナ内で定義されていない場合でも、Slimはそれを構築し、最初の引数としてコンテナをコンストラクタに渡します！
$app->get('/', Home::class . ':index');


// In container.config.php
// ここではSlimTwigを使用します。
return [
    'settings' => [
        'viewTemplatesDirectory' => '../templates',
    ],
    'twig' => [
        'title' => '',
        'description' => '',
        'author' => ''
    ],
    'view' => function ($c) {
        $view = new Twig(
            $c['settings']['viewTemplatesDirectory'],
            [
                'cache' => false // '../cache'
            ]
        );
        
        // インスタンス化とSlim固有の拡張機能
        $view->addExtension(
            new TwigExtension(
                $c['router'],
                $c['request']->getUri()
            )
        );

        foreach ($c['twig'] as $name => $value) {
            $view->getEnvironment()->addGlobal($name, $value);
        }

        return $view;
    },
    Home::class => function ($c) {
        return new Home($c['view']);
    }
];

```

## PSR-7 Objects

### リクエスト、レスポンス、URI、アップロードファイルは変更できません。
これらのオブジェクトのいずれかを変更しても、古いインスタンスは更新されないことを意味します。

```php
// これは間違いです。この変更は適用されません。
$app->add(function (Request $request, Response $response, $next) {
    $request->withAttribute('abc', 'def');
    return $next($request, $response);
});

// こちらが正解です。
$app->add(function (Request $request, Response $response, $next) {
    $request = $request->withAttribute('abc', 'def');
    return $next($request, $response);
});
```

### Message bodies are streams

```php
// ...
$image = __DIR__ . '/huge_photo.jpg';
$body = new Stream($image);
$response = (new Response())
     ->withStatus(200, 'OK')
     ->withHeader('Content-Type', 'image/jpeg')
     ->withHeader('Content-Length', filesize($image))
     ->withBody($body);
// ...
```

文字列の場合
```php
// ...
$response = (new Response())->getBody()->write('Hello world!')

// またはSlim固有の方法。PSR-7には準拠していません。
$response = (new Response())->write('Hello world!');
// ...
```
