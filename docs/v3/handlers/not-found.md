---
title: 404 Not Found Handler
---

Slim FrameworkアプリケーションにHTTPリクエストURIと一致するルートがない場合、アプリケーションはNot Foundハンドラーを呼び出し、HTTPクライアントに`HTTP/1.1 404 Not Found`レスポンスを返します。

## Default Not Found handler

Slim FrameworkアプリケーションのNot FoundハンドラーはPimpleサービスです。 Appオブジェクトをインスタンス化する前に、アプリケーションコンテナーでカスタムPimpleファクトリメソッドを定義することにより、独自のNot Foundハンドラーに置き換えることができます。

各Slim FrameworkアプリケーションにはデフォルトのNot Foundハンドラーがあります。このハンドラーは、レスポンスステータスを`404`に、コンテンツタイプを`text/html`に設定し、レスポンスボディに簡単な説明を書き込みます。

## Custom Not Found handler

Slim FrameworkアプリケーションのNot FoundハンドラーはPimpleサービスです。 Appオブジェクトをインスタンス化する前に、アプリケーションコンテナーでカスタムPimpleファクトリメソッドを定義することにより、独自のNot Foundハンドラーに置き換えることができます。

```php
$c = new \Slim\Container(); //Create Your container

//Override the default Not Found Handler before creating App
$c['notFoundHandler'] = function ($c) {
    return function ($request, $response) use ($c) {
        return $response->withStatus(404)
            ->withHeader('Content-Type', 'text/html')
            ->write('Page not found');
    };
};

//Create Slim
$app = new \Slim\App($c);

//... Your code
```

この例では、callbleオブジェクトを返す新しい`notFoundHandler`ファクトリーを定義します。返却されるcallableは2つの引数を受け入れます。

1. `\Psr\Http\Message\ServerRequestInterface`インスタンス
2. `\Psr\Http\Message\ResponseInterface`インスタンス

callableは適切な`\Psr\Http\Message\ResponseInterface`インスタンスを返さなければなりません。

Appオブジェクトをインスタンス化した後にデフォルトのNot Foundハンドラーをオーバーライドしたい場合は、デフォルトハンドラーの設定を`unset`メソッドで解除してから上書きできます。

```php
$c = new \Slim\Container(); //Create Your container

//Create Slim
$app = new \Slim\App($c);

//... Your code

//Override the default Not Found Handler after App
unset($app->getContainer()['notFoundHandler']);
$app->getContainer()['notFoundHandler'] = function ($c) {
    return function ($request, $response) use ($c) {
        $response = new \Slim\Http\Response(404);
        return $response->write("Page not found");
    };
};
```
