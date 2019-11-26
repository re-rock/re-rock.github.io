---
title: Retrieving Current Route
---

アプリケーション内の現在のルートへのアクセスを取得する必要がある場合、`'route'`引数を指定したリクエストクラスの`getAttribute`メソッドを呼び出すだけで、現在のルート（`Slim\Route`クラスのインスタンス）が返されます。

そこから、`getName()`を使用してルートの名前の取得や、`getMethods()`などで、ルートでサポートされているメソッドを取得できます。

注：アプリのミドルウェア内からルートにアクセスする必要がある場合、ミドルウェア構成箇所で`'determineRouteBeforeAppMiddleware'`をtrueに設定する必要があります。
そうしないと、`getAttribute('route')`はnullを返します。また`getAttribute('route')`は存在しないルートでもnullを返します。

Example:
```php
use Slim\App;
use Slim\Exception\NotFoundException;
use Slim\Http\Request;
use Slim\Http\Response;

$app = new App([
    'settings' => [
        // Only set this if you need access to route within middleware
        'determineRouteBeforeAppMiddleware' => true
    ]
]);

// routes...
$app->add(function (Request $request, Response $response, callable $next) {
    $route = $request->getAttribute('route');

    // return NotFound for non existent route
    if (empty($route)) {
        throw new NotFoundException($request, $response);
    }

    $name = $route->getName();
    $groups = $route->getGroups();
    $methods = $route->getMethods();
    $arguments = $route->getArguments();

    // do something with that information

    return $next($request, $response);
});
```
