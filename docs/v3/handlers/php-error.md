---
title: PHP Error Handler
---

Slimフレームワークアプリケーションが[PHP Runtime error](http://php.net/manual/en/class.error.php)（PHP7+）をスローした場合、アプリケーションはPHPエラーハンドラーを呼び出し、HTTPクライアントに`HTTP/1.1 500 Internal Server Error`レスポンスを返します。

`Warnings`と`Notices`はデフォルトではキャッチされないことに注意してください。それらがが発生したときにアプリケーションでエラーページを表示したい場合は、アプリを起動する前に次のコードを実行する必要があります。ほとんどの場合、これは`index.php`の最上部に追加することを意味します。

```php
<?php

error_reporting(E_ALL);
set_error_handler(function ($severity, $message, $file, $line) {
    if (error_reporting() & $severity) {
        throw new \ErrorException($message, 0, $severity, $file, $line);
    }
});
```

これにより、すべての`Warnings`と`Notices`が`Errors`として扱われ、定義されたハンドラーが発動します。

## Default PHP Error handler

各Slim Frameworkアプリケーションには、デフォルトのPHPエラーハンドラーがあります。このハンドラーは、レスポンスステータスを`500`に、コンテンツタイプを`text/html`に設定し、簡単な説明をレスポンスボディに書き込みます。

## Custom PHP Error handler

Slim FrameworkアプリケーションのPHPエラーハンドラーはPimpleサービスです。アプリケーションコンテナーでカスタムPimpleファクトリメソッドを定義することにより、独自のPHPエラーハンドラーに置き換えることができます。

```php
// Create Slim
$app = new \Slim\App();
// get the app's di-container
$c = $app->getContainer();
$c['phpErrorHandler'] = function ($c) {
    return function ($request, $response, $error) use ($c) {
        return $response->withStatus(500)
            ->withHeader('Content-Type', 'text/html')
            ->write('Something went wrong!');
    };
};
```

> 備考 `\Slim\Container`の新しいインスタンスを使用したpre-slim作成メソッドは[Not Found](/docs/v3/handlers/not-found.html)ドキュメントで確認してください

この例では、callableオブジェクトを返す新しい`phpErrorHandler`ファクトリーを定義します。返されるcallableは3つの引数を受け入れます。

1. `\Psr\Http\Message\ServerRequestInterface`インスタンス
2. `\Psr\Http\Message\ResponseInterface`インスタンス
3. `\Throwable`インスタンス

callableオブジェクトは、与えられたエラーに適した新しい`\Psr\Http\Message\ResponseInterface`インスタンスを返さなければなりません。
