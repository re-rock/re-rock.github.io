---
title: Slim 3 Documentation
---

<div class="alert alert-info">
    <p>
        このドキュメントは<strong>Slim 3</strong>向けです。Slim4をお使いの場合は<a href="/docs/v4">こちら</a>
    </p>
</div>

## Welcome

SlimフレームワークはWebアプリとAPIを素早くかつ強力に作成することができるマイクロフレームワークです。  
その中心となるものは、HTTPリクエスト、割り当てられたコールバックルーティンの発動、HTTPレスポンスの 
返却を管理することです。

## What's the point?

Slimフレームワークはデータの使用、再利用、公開を行うAPIを作成する理想的なツールです。 
また試作品を作ることにも適しています。またユーザーインターフェースを備えたフル機能の 
Webアプリを作成することもできます。さらに重要なことはSlimは極めて軽く少量のコードで
作成されています。事実、午後だけでソースコードを読んで理解することができるでしょう。

> その中心となるものは、HTTPリクエスト、割り当てられたコールバックルーティンの発動、HTTPレスポンスの 
  返却を管理することです。

常にSymfonyやLaravellのようなフルスタックフレームワークは必要ないでしょう。確かにそれらは素晴らしいツールですが 
ケースによっては機能が多すぎます。そのかわりSlimは必要とする最低限の機能を提供するだけです。

## How does it work?

最初にApacheやNginXなどのWebサーバが必要です。フロントコントローラーの役割を行うPHPファイルに送信される全ての  
リクエストを受け取るため [Webサーバの設定](/docs/v3/start/web-servers.html)を行ってください。このファイルで
Slimアプリのインスタンス化と実行が行われます。

Slimアプリには特定のHTTPリクエストに応答するルータが含まれています。
各ルータはコールバックを呼び出し、HTTPレスポンスの返却します。まず最初にSlimアプリのインスタンス化と設定を行います。 
次にアプリケーションルートを定義します。最後にアプリを実行します。簡単ですね。以下に例を示します。

<figure markdown="1">
```php
<?php
// Create and configure Slim app
$config = ['settings' => [
    'addContentLengthHeader' => false,
]];
$app = new \Slim\App($config);

// Define app routes
$app->get('/hello/{name}', function ($request, $response, $args) {
    return $response->write("Hello " . $args['name']);
});

// Run app
$app->run();
```
<figcaption>Figure 1: Example Slim application</figcaption>
</figure>

## Request and response

When you build a Slim app, you are often working directly with Request
and Response objects. These objects represent the actual HTTP request received
by the web server and the eventual HTTP response returned to the client.

Every Slim app route is given the current Request and Response objects as arguments
to its callback routine. These objects implement the popular [PSR-7](/docs/v3/concepts/value-objects.html) interfaces. The Slim app route can inspect
or manipulate these objects as necessary. Ultimately, each Slim app route
**MUST** return a PSR-7 Response object.

## Bring your own components

Slim is designed to play well with other PHP components, too. You can register
additional first-party components such as [Slim-Csrf][csrf], [Slim-HttpCache][httpcache],
or [Slim-Flash][flash] that build upon Slim's default functionality. It's also
easy to integrate third-party components found on [Packagist](https://packagist.org/).

## How to read this documentation

If you are new to Slim, I recommend you read this documentation from start
to finish. If you are already familiar with Slim, you can instead jump straight
to the appropriate section.

This documentation begins by explaining Slim's concepts and architecture
before venturing into specific topics like request and response handling,
routing, and error handling.

## Documentation License
<p style="text-align: left;">
    This website and documentation is licensed under a <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>.
    <br />
    <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">
        <img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" />
    </a>
</p>

[symfony]: https://symfony.com/
[laravel]: https://laravel.com/
[csrf]: https://github.com/slimphp/Slim-Csrf/
[httpcache]: https://github.com/slimphp/Slim-HttpCache
[flash]: https://github.com/slimphp/Slim-Flash
[eloquent]: https://laravel.com/docs/5.1/eloquent
[doctrine]: http://www.doctrine-project.org/projects/orm.html
