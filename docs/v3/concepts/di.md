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

Add a service to Slim container:

```php
$container = $app->getContainer();
$container['myService'] = function ($container) {
    $myService = new MyService();
    return $myService;
};
```

You can fetch services from your container explicitly or implicitly.
You can fetch an explicit reference to the container instance from inside a Slim
application route like this:

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

You can implicitly fetch services from the container like this:

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

To test if a service exists in the container before using it, use the `has()` method, like this:

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


Slim uses `__get()` and `__isset()` magic methods that defer to the application's
container for all properties that do not already exist on the application instance.

## Required services

Your container MUST implement these required services. If you use Slim's built-in container, these are provided for you. If you choose a third-party container, you must define these required services on your own.

settings
:   Associative array of application settings, including keys:
    
    * `httpVersion`
    * `responseChunkSize`
    * `outputBuffering`
    * `determineRouteBeforeAppMiddleware`.
    * `displayErrorDetails`.
    * `addContentLengthHeader`.
    * `routerCacheFile`.

environment
:   Instance of `\Slim\Http\Environment`.

request
:   Instance of `\Psr\Http\Message\ServerRequestInterface`.

response
:   Instance of `\Psr\Http\Message\ResponseInterface`.

router
:   Instance of `\Slim\Interfaces\RouterInterface`.

foundHandler
:   Instance of `\Slim\Interfaces\InvocationStrategyInterface`.

phpErrorHandler
:   Callable invoked if a PHP 7 Error is thrown. The callable **MUST** return an instance of `\Psr\Http\Message\ResponseInterface` and accept three arguments:

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`
3. `\Error`

errorHandler
:   Callable invoked if an Exception is thrown. The callable **MUST** return an instance of `\Psr\Http\Message\ResponseInterface` and accept three arguments:

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`
3. `\Exception`

notFoundHandler
:   Callable invoked if the current HTTP request URI does not match an application route. The callable **MUST** return an instance of `\Psr\Http\Message\ResponseInterface` and accept two arguments:

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`

notAllowedHandler
:   Callable invoked if an application route matches the current HTTP request path but not its method. The callable **MUST** return an instance of `\Psr\Http\Message\ResponseInterface` and accept three arguments:

1. `\Psr\Http\Message\ServerRequestInterface`
2. `\Psr\Http\Message\ResponseInterface`
3. Array of allowed HTTP methods

callableResolver
:   Instance of `\Slim\Interfaces\CallableResolverInterface`.
