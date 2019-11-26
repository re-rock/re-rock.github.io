---
title: 405 Not Allowed Handler
---

Slimフレームワークアプリケーションで、HTTPリクエストURIには一致するがHTTPリクエストメソッドに一致しないルートがあった場合、アプリケーションはNot Allowedハンドラーを呼び出し、HTTPクライアントに `HTTP/1.1 405 Not Allowed`レスポンスを返します。

## Default Not Allowed handler

各Slim Frameworkアプリケーションには、デフォルトのNot Allowedハンドラーがあります。このハンドラーは、レスポンスステータスを`405`に、コンテンツタイプを`text/html`に設定し、
許可されているHTTPメソッドのカンマ区切りのリストとともに`Allowed:`をHTTPヘッダーに追加し、簡単な説明をレスポンスボディに書き込みます。

## Custom Not Allowed handler

Slim FrameworkアプリケーションのNot Allowedハンドラーは、Pimpleサービスです。アプリケーションコンテナーでカスタムPimpleファクトリメソッドを定義することにより、独自のNot Allowedハンドラーに置き換えることができます。

```php
// Create Slim
$app = new \Slim\App();
// get the app's di-container
$c = $app->getContainer();
$c['notAllowedHandler'] = function ($c) {
    return function ($request, $response, $methods) use ($c) {
        return $response->withStatus(405)
            ->withHeader('Allow', implode(', ', $methods))
            ->withHeader('Content-type', 'text/html')
            ->write('Method must be one of: ' . implode(', ', $methods));
    };
};
```

> 備考 `\Slim\Container`の新しいインスタンスを使用したpre-slim作成メソッドは[Not Found](/docs/v3/handlers/not-found.html)ドキュメントで確認してください

この例では、callbleオブジェクトを返す新しい`notAllowedHandler`ファクトリーを定義します。返されるcallableは3つの引数を受け入れます。

1. `\Psr\Http\Message\ServerRequestInterface`インスタンス
2. `\Psr\Http\Message\ResponseInterface`インスタンス
3. 許可されるHTTPメソッド名を要素に持つ配列

callableは適切な`\Psr\Http\Message\ResponseInterface`インスタンスを返さなければなりません。
