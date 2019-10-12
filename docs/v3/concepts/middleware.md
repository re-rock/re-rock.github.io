---
title: Middleware
---

必要に応じてSlimアプリケーションの実行前後に、RequestオブジェクトとResponseオブジェクトを操作できます。
これをミドルウェアと呼びます。なぜこれが必要でしょうか。たとえばクロスサイトリクエストフォージェリから
アプリを保護する必要がある場合だったり、アプリを実行する前にリクエストを認証したい場合などです。
ミドルウェアはこれらのケースにとても有用です。

## What is middleware?

技術的に言えば、ミドルウェアは3つの引数を受け入れるcallableオブジェクトです。

1. `\Psr\Http\Message\ServerRequestInterface` - PSR7 リクエストオブジェクト
2. `\Psr\Http\Message\ResponseInterface` - PSR 7レスポンスオブジェクト
3. `callable` - 新たなミドルウェアcallable

これらのオブジェクトに則していれば、いろんなことができます。
絶対条件は、ミドルウェアが`\Psr\Http\Message\ResponseInterface`のインスタンスを**必ず**返さなければならないことです。
また各ミドルウェアが次のミドルウェアを呼び出す際に、RequestおよびResponseオブジェクトを引数として渡すことが推奨されます。

## How does middleware work?

フレームワークが異なれば、ミドルウェアの使用方法も異なります。 
Slimはミドルウェアをコアアプリケーションを囲む同心円層として追加しています。新しいミドルウェア層はそれぞれ
既存のミドルウェア層を囲んでいます。同心円構造は、ミドルウェアレイヤーが追加されると外側に拡張します。

最後に追加されたミドルウェア層が最初に実行されます。

Slimアプリケーションを実行すると、RequestオブジェクトとResponseオブジェクトは外部からミドルウェア構造を通過します。
最初にもっとも外側のミドルウェアを通過した後、次に外側のミドルウェアを通過するという様に繰り返し、最終的にSlimアプリケーション本体に到着します。

Slimアプリケーションが対象のルートに処理を送り出した後、Slimアプリケーションは処理結果のResponseオブジェクトを保持しながら、
ミドルウェア構造を内側から外側へと横断します。
最終的にResponseオブジェクトは、もっとも外側のミドルウェアを出て、生のHTTPレスポンスにシリアル化され、
HTTPクライアントに返されます。ミドルウェアのプロセスフローを示す図を次に示します。

<div style="padding: 2em 0; text-align: center">
    <img src="/docs/v3/images/middleware.png" alt="Middleware architecture" style="max-width: 80%;"/>
</div>

## How do I write middleware?

ミドルウェアは、リクエストオブジェクト、レスポンスオブジェクト、および次のミドルウェアオブジェクトの3つの引数を受け入れる、callbleオブジェクトです。各ミドルウェアは、`\Psr\Http\Message\ResponseInterface`のインスタンスを**必ず**返さなければなりません。

### Closure middleware example.

このサンプルのミドルウェアはクロージャーです。

```php
<?php
/**
 * Example middleware closure
 *
 * @param  \Psr\Http\Message\ServerRequestInterface $request  PSR7 request
 * @param  \Psr\Http\Message\ResponseInterface      $response PSR7 response
 * @param  callable                                 $next     Next middleware
 *
 * @return \Psr\Http\Message\ResponseInterface
 */
function ($request, $response, $next) {
    $response->getBody()->write('BEFORE');
    $response = $next($request, $response);
    $response->getBody()->write('AFTER');

    return $response;
};
```

### Invokable class middleware example

このサンプルのミドルウェアは、`__invoke()`メソッドを実装する呼び出し可能クラスです。

This example middleware is an invokable class that implements the magic `__invoke()` method.

```php
<?php
class ExampleMiddleware
{
    /**
     * Example middleware invokable class
     *
     * @param  \Psr\Http\Message\ServerRequestInterface $request  PSR7 request
     * @param  \Psr\Http\Message\ResponseInterface      $response PSR7 response
     * @param  callable                                 $next     Next middleware
     *
     * @return \Psr\Http\Message\ResponseInterface
     */
    public function __invoke($request, $response, $next)
    {
        $response->getBody()->write('BEFORE');
        $response = $next($request, $response);
        $response->getBody()->write('AFTER');

        return $response;
    }
}
```

このクラスをミドルウェアとして使用する場合、以下のコードのように`$subject`として表すことができる`$app`、`Route`、または`group()`の後に`->add(new ExampleMiddleware());`メソッドを使用できます。

```php
$subject->add( new ExampleMiddleware() );
```

## How do I add middleware?

ミドルウェアは、Slimアプリケーションの別々のアプリケーションルート、またはルートグループに追加できます。すべてのシナリオで同じミドルウェアを受け入れ、同じミドルウェアインターフェイスを実装します。

### Application middleware

アプリケーションミドルウェアは、受信したHTTPリクエストごとに呼び出されます。 Slimアプリケーションインスタンスの`add()`メソッドでアプリケーションミドルウェアを追加します。この例では、上記の例で紹介したClosureミドルウェアの追加しています。

```php
<?php
$app = new \Slim\App();

$app->add(function ($request, $response, $next) {
	$response->getBody()->write('BEFORE');
	$response = $next($request, $response);
	$response->getBody()->write('AFTER');

	return $response;
});

$app->get('/', function ($request, $response, $args) {
	$response->getBody()->write(' Hello ');

	return $response;
});

$app->run();
```

この場合、次のHTTPレスポンス本文が出力されます。

This would output this HTTP response body:

    `BEFORE Hello AFTER`

### Route middleware

Route middleware is invoked _only if_ its route matches the current HTTP request method and URI. Route middleware is specified immediately after you invoke any of the Slim application's routing methods (e.g., `get()` or `post()`). Each routing method returns an instance of `\Slim\Route`, and this class provides the same middleware interface as the Slim application instance. Add middleware to a Route with the Route instance's `add()` method. This example adds the Closure middleware example above:

```php
<?php
$app = new \Slim\App();

$mw = function ($request, $response, $next) {
    $response->getBody()->write('BEFORE');
    $response = $next($request, $response);
    $response->getBody()->write('AFTER');

    return $response;
};

$app->get('/', function ($request, $response, $args) {
	$response->getBody()->write(' Hello ');

	return $response;
})->add($mw);

$app->run();
```

This would output this HTTP response body:

    BEFORE Hello AFTER

### Group Middleware

アプリケーション全体、およびミドルウェアを受け入れることができる標準ルートに加えて、`group()`マルチルート定義機能は、個々のルートを内部的に許可します。
ルートグループミドルウェアは、そのルートがグループから定義されたHTTPリクエストメソッドおよびURIのいずれかに一致する場合にのみ呼び出されます。 
コールバック内にミドルウェアを追加し、`group()`メソッドの後に`add()`をチェーンして設定するグループ全体のミドルウェアを追加します。

以下はURLハンドラーのグループでコールバックミドルウェアを使用するサンプルアプリケーションです。
```php
<?php

require_once __DIR__.'/vendor/autoload.php';

$app = new \Slim\App();

$app->get('/', function ($request, $response) {
    return $response->getBody()->write('Hello World');
});

$app->group('/utils', function () use ($app) {
    $app->get('/date', function ($request, $response) {
        return $response->getBody()->write(date('Y-m-d H:i:s'));
    });
    $app->get('/time', function ($request, $response) {
        return $response->getBody()->write(time());
    });
})->add(function ($request, $response, $next) {
    $response->getBody()->write('It is now ');
    $response = $next($request, $response);
    $response->getBody()->write('. Enjoy!');

    return $response;
});
```
`/utils/date`メソッドを呼び出すと、次のような文字列が出力されます

    `It is now 2015-07-06 03:11:01. Enjoy!`

`/utils/time`にアクセスすると、次のような文字列が出力されます

    `It is now 1436148762. Enjoy!`

しかし`/`*(domain-root)*にアクセスすると、ミドルウェアが割り当てられていないため、次の出力が生成されます。

    `Hello World`

### Passing variables from middleware
ミドルウェアから属性を渡すもっとも簡単な方法は、リクエストの属性を使用することです。

ミドルウェアで変数を設定する：
```php
$request = $request->withAttribute('foo', 'bar');
```

ルートコールバックで変数を取得する：

```php
$foo = $request->getAttribute('foo');
```

## Finding available middleware

ニーズに合ったPSR-7ミドルウェアクラスがすでに作成されている場合があります。非公式ですが参考になるリストを次に示します。

* [oscarotero/psr7-middlewares](https://github.com/oscarotero/psr7-middlewares)
* [Middleware for Slim Framework v3.x wiki](https://github.com/slimphp/Slim/wiki/Middleware-for-Slim-Framework-v3.x)
* [lalop/awesome-psr7](https://github.com/lalop/awesome-psr7)
