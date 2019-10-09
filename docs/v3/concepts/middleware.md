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

Different frameworks use middleware differently. Slim adds middleware as concentric
layers surrounding your core application. Each new middleware layer surrounds
any existing middleware layers. The concentric structure expands outwardly as
additional middleware layers are added.

The last middleware layer added is the first to be executed.

When you run the Slim application, the Request and Response objects traverse the
middleware structure from the outside in. They first enter the outer-most middleware,
then the next outer-most middleware, (and so on), until they ultimately arrive
at the Slim application itself. After the Slim application dispatches the
appropriate route, the resultant Response object exits the Slim application and
traverses the middleware structure from the inside out. Ultimately, a final
Response object exits the outer-most middleware, is serialized into a raw HTTP
response, and is returned to the HTTP client. Here's a diagram that illustrates
the middleware process flow:

<div style="padding: 2em 0; text-align: center">
    <img src="/docs/v3/images/middleware.png" alt="Middleware architecture" style="max-width: 80%;"/>
</div>

## How do I write middleware?

Middleware is a callable that accepts three arguments: a Request object, a Response object, and the next middleware. Each middleware **MUST** return an instance of `\Psr\Http\Message\ResponseInterface`.

### Closure middleware example.

This example middleware is a Closure.

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

To use this class as a middleware, you can use `->add( new ExampleMiddleware() );` function chain after the `$app`, `Route`,  or `group()`, which in the code below, any one of these, could represent $subject.

```php
$subject->add( new ExampleMiddleware() );
```

## How do I add middleware?

You may add middleware to a Slim application, to an individual Slim application route or to a route group. All scenarios accept the same middleware and implement the same middleware interface.

### Application middleware

Application middleware is invoked for every *incoming* HTTP request. Add application middleware with the Slim application instance's `add()` method. This example adds the Closure middleware example above:

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

This would output this HTTP response body:

    BEFORE Hello AFTER

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

In addition to the overall application, and standard routes being able to accept middleware, the `group()` multi-route definition functionality, also allows individual routes internally. Route group middleware is invoked _only if_ its route matches one of the defined HTTP request methods and URIs from the group. To add middleware within the callback, and entire-group middleware to be set by chaining `add()` after the `group()` method.

Sample Application, making use of callback middleware on a group of url-handlers
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

When calling the `/utils/date` method, this would output a string similar to the below

    It is now 2015-07-06 03:11:01. Enjoy!

visiting `/utils/time` would output a string similar to the below

    It is now 1436148762. Enjoy!

but visiting `/` *(domain-root)*, would be expected to generate the following output as no middleware has been assigned

    Hello World

### Passing variables from middleware
The easiest way to pass attributes from middleware is to use the request's
attributes.

Setting the variable in the middleware:

```php
$request = $request->withAttribute('foo', 'bar');
```

Getting the variable in the route callback:

```php
$foo = $request->getAttribute('foo');
```

## Finding available middleware

You may find a PSR-7 Middleware class already written that will satisfy your needs. Here are a few unofficial lists to search.

* [oscarotero/psr7-middlewares](https://github.com/oscarotero/psr7-middlewares)
* [Middleware for Slim Framework v3.x wiki](https://github.com/slimphp/Slim/wiki/Middleware-for-Slim-Framework-v3.x)
* [lalop/awesome-psr7](https://github.com/lalop/awesome-psr7)
