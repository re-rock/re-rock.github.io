---
title: PSR-7 and Value Objects
---

Slimのレクエストおよびレスポンスオブジェクトは[PSR-7](https://github.com/php-fig/http-message) インターフェイスをサポートします。そのためPSR-7の実装を使用でき柔軟に実装できます。たとえばSlimアプリケーションルートは`\Slim\Http\Response`インスタンスを返す必要はありません。
`\GuzzleHttp\Psr7\CachingStream`のインスタンスや`\GuzzleHttp\Psr7\stream_for()`関数によって返されたインスタンスなどを返すことができます。

Slimは簡単に利用できるよう独自のPSR-7実装を提供しています。ただし、SlimのデフォルトのPSR-7オブジェクトをサードパーティの実装に自由に置き換えることもできます。アプリケーションコンテナのリクエストおよびレスポンスサービスをオーバーライドするだけで、`\Psr\Http\Message\ServerRequestInterface`および`\Psr\Http\Message\ResponseInterface`のインスタンスをそれぞれ返します。

## Value objects

SlimのRequestおよびResponseオブジェクトは[不変オブジェクト](http://en.wikipedia.org/wiki/Value_object)です。プロパティ値をアップデートするためのクローンバージョンをリクエストすることによってのみ変更できます。バリューオブジェクトは、プロパティの更新時に複製する必要があるため、わずかなオーバーヘッドがあります。ただこのオーバーヘッドがパフォーマンスに影響を与えることはありません。

バリューオブジェクトのコピーをレクエストするには、PSR-7インターフェイスメソッドを呼び出します（これらのメソッドには通常`with`プレフィックスが付いています）。たとえば、PSR-7レスポンスオブジェクトには、新しいHTTPヘッダーを持つクローンバリューオブジェクトを返す`withHeader($name、$value)`メソッドがあります。

```php
<?php
$app = new \Slim\App;
$app->get('/foo', function ($req, $res, $args) {
    return $res->withHeader(
        'Content-Type',
        'application/json'
    );
});
$app->run();
```

PSR-7インターフェースは、リクエストオブジェクトとレスポンスオブジェクトを変換するために次のメソッドを提供します。

* `withProtocolVersion($version)`
* `withHeader($name, $value)`
* `withAddedHeader($name, $value)`
* `withoutHeader($name)`
* `withBody(StreamInterface $body)`

PSR-7インターフェースは、リクエストオブジェクトを変換するために次のメソッドを提供します。

* `withMethod($method)`
* `withUri(UriInterface $uri, $preserveHost = false)`
* `withCookieParams(array $cookies)`
* `withQueryParams(array $query)`
* `withUploadedFiles(array $uploadedFiles)`
* `withParsedBody($data)`
* `withAttribute($name, $value)`
* `withoutAttribute($name)`

PSR-7インターフェイスは、レスポンスオブジェクトを変換するために次のメソッドを提供します。

* `withStatus($code, $reasonPhrase = '')`

これらの方法の詳細については[PSR-7ドキュメント](http://www.php-fig.org/psr/psr-7/)を参照してください。
