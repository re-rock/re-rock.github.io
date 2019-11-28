---
title: Setting up CORS
---

CORS - Cross origin resource sharing

CORSサポートを実装するための最適なフローチャート
[CORS server flowchart](http://www.html5rocks.com/static/images/cors_server_flowchart.png)

ここでCORSサポートをテストできます : http://www.test-cors.org/

仕様はこちらで読むことができます : https://www.w3.org/TR/cors/


## The simple solution

シンプルなCORSリクエストの場合、サーバーはそのレスポンスに次のヘッダーを追加するだけです。

`Access-Control-Allow-Origin: <domain>, ... | *`

次のコードは、lazy CORSを有効にする必要があります。

```php
$app->options('/{routes:.+}', function ($request, $response, $args) {
    return $response;
});

$app->add(function ($req, $res, $next) {
    $response = $next($req, $res);
    return $response
            ->withHeader('Access-Control-Allow-Origin', 'http://mysite')
            ->withHeader('Access-Control-Allow-Headers', 'X-Requested-With, Content-Type, Accept, Origin, Authorization')
            ->withHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS');
});
```

最後のルートとして下記のルート追加します。

```php
// 一致するルートがない場合、すべてのルートが404 Not Foundページに誘導されます
// 注：このルートは最後に定義することを確認してください
$app->map(['GET', 'POST', 'PUT', 'DELETE', 'PATCH'], '/{routes:.+}', function($req, $res) {
    $handler = $this->notFoundHandler; // handle using the default Slim page not found handler
    return $handler($req, $res);
});
```


## Access-Control-Allow-Methods

次のミドルウェアを使用して、Slimのルーターへクエリを行い、特定のパターンが実装するメソッドのリストを取得できます。
下記はサンプルアプリケーションです

```php
require __DIR__ . "/vendor/autoload.php";

// このSlimの設定は、ミドルウェアを機能させるために必要です
$app = new Slim\App([
    "settings"  => [
        "determineRouteBeforeAppMiddleware" => true,
    ]
]);

// これはミドルウェアです
// Access-Control-Allow-Methodsヘッダーをすべてのリクエストに追加します
$app->add(function($request, $response, $next) {
    $route = $request->getAttribute("route");

    $methods = [];

    if (!empty($route)) {
        $pattern = $route->getPattern();

        foreach ($this->router->getRoutes() as $route) {
            if ($pattern === $route->getPattern()) {
                $methods = array_merge_recursive($methods, $route->getMethods());
            }
        }
        // メソッドは、特定のルートが処理するすべてのHTTP動詞を保持します。
    } else {
        $methods[] = $request->getMethod();
    }

    $response = $next($request, $response);


    return $response->withHeader("Access-Control-Allow-Methods", implode(",", $methods));
});

$app->get("/api/{id}", function($request, $response, $arguments) {
});

$app->post("/api/{id}", function($request, $response, $arguments) {
});

$app->map(["DELETE", "PATCH"], "/api/{id}", function($request, $response, $arguments) {
});

// なにかしらjavascriptフロントエンドフレームワークを使用している場合、Slimでgroupを使用する場合は注意してください
$app->group('/api', function () {
  　// ブラウザの規定動作のため、PUTまたはDELETEリクエストを送信するときはOPTIONSメソッドを追加する必要があります。詳細はpreflightについてお読みください。
    $this->map(['PUT', 'OPTIONS'], '/{user_id:[0-9]+}', function ($request, $response, $arguments) {
        // Your code here...
    });
});

$app->run();
```

この記事を提案してくれた[tuupola](https://github.com/tuupola) に感謝します！
