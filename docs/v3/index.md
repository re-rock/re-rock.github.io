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
// Slimアプリの作成と構成
$config = ['settings' => [
    'addContentLengthHeader' => false,
]];
$app = new \Slim\App($config);

// ルートの定義
$app->get('/hello/{name}', function ($request, $response, $args) {
    return $response->write("Hello " . $args['name']);
});

// アプリの実行
$app->run();
```
<figcaption>Figure 1: Example Slim application</figcaption>
</figure>

## Request and response

Slimアプリをビルドするときは通常リクエストオブジェクトとレスポンスオブジェクトを直接操作します。
これらのオブジェクトは、Webサーバーが受信した実際のHTTPリクエストと、クライアントに返される最終的なHTTPレスポンスを表します。 
すべてのSlimアプリルートには、コールバックルーティンへの引数として最新のリクエストおよびレスポンスオブジェクトが与えられます。
これらのオブジェクトは、一般的な[PSR-7インターフェイス](/docs/v3/concepts/value-objects.html) を実装しています。

Slimアプリルートは、必要に応じてこれらのオブジェクトを検査または操作できます。
最終的には各SlimアプリルータはPSR-7レスポンスオブジェクトを**必ず**返さなければなりません。

## Bring your own components

Slimはまた他のPHPコンポーネントともうまく連携するように設計されています。
Slimの標準機能に基づいて構築された[Slim-Csrf][csrf]、[Slim-HttpCache][httpcache]、[Slim-Flash][flash]などの
追加のファーストパーティコンポーネントを登録できます。また、[Packagist](https://packagist.org/)にあるサードパーティの
コンポーネントを簡単に統合できます。

## How to read this documentation

Slimを初めて使用する場合は、このドキュメントを最初から最後まで読むことをお勧めします。すでにSlimに慣れている場合は、
知りたいセクションを直接お読みください。  
このドキュメントではリクエストとレスポンスの処理、ルーティング、エラー処理などの特定のトピックに進む前に
Slimの概念とアーキテクチャを説明します。

## Documentation License
<p style="text-align: left;">
    このWebサイトおよびドキュメントは以下のライセンスに属しています。  

    <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>
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
