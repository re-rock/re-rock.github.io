---
title: Retrieving IP address
---

クライアントの現在のIPアドレスを取得する最良の方法は、[ip-address-middleware](https://github.com/akrabat/ip-address-middleware/)などのコンポーネントを使用するミドルウェアを使用することです。

このコンポーネントは、composerを介してインストールできます。

```bash
composer require akrabat/ip-address-middleware
```

プロキシ環境での使い方は、信頼できるプロキシ（varnishサーバーなど）のリストを、<code>App</code>内のミドルウェアに登録します。

```php
$checkProxyHeaders = true;
$trustedProxies = ['10.0.0.1', '10.0.0.2'];
$app->add(new RKA\Middleware\IpAddress($checkProxyHeaders, $trustedProxies));

$app->get('/', function ($request, $response, $args) {
    $ipAddress = $request->getAttribute('ip_address');

    return $response;
});
```

ミドルウェアはクライアントのIPアドレスをリクエスト属性に保存するため、アクセスは<code>$request->getAttribute('ip_address')</code>を介して行われます。
