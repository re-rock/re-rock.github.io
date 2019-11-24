---
title: Response
---

Slimアプリのルートとミドルウェアには、クライアントに返却されるHTTPレスポンスを表すPSR-7レスポンスオブジェクトが与えられます。
レスポンスオブジェクトは、HTTPレスポンスステータス、ヘッダー、およびボディを検査および操作できる [PSR-7 ResponseInterface][psr7] を実装します。

[psr7]: http://www.php-fig.org/psr/psr-7/#3-2-1-psr-http-message-responseinterface

## How to get the Response object

PSR-7レスポンスオブジェクトは、次のようにルートコールバックの2番目の引数としてSlimアプリケーションルートに挿入されます。

<figure markdown="1">
```php
<?php
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

$app = new \Slim\App;
$app->get('/foo', function (ServerRequestInterface $request, ResponseInterface $response) {
    // Use the PSR-7 $response object

    return $response;
});
$app->run();
```
<figcaption>Figure 1: Inject PSR-7 response into application route callback.</figcaption>
</figure>

PSR-7レスポンスオブジェクトは、次のように呼び出し可能なミドルウェアの2番目の引数としてSlimアプリケーションミドルウェアに注入されます。

The PSR-7 response object is injected into your Slim application _middleware_
as the second argument of the middleware callable like this:

<figure markdown="1">
```php
<?php
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

$app = new \Slim\App;
$app->add(function (ServerRequestInterface $request, ResponseInterface $response, callable $next) {
    // Use the PSR-7 $response object

    return $next($request, $response);
});
// Define app routes...
$app->run();
```
<figcaption>Figure 2: Inject PSR-7 response into application middleware.</figcaption>
</figure>

## The Response Status

すべてのHTTPレスポンスには、数値型の [status code][statuscodes] があります。ステータスコードは、クライアントに返されるHTTPレスポンスの _type_ を識別します。
PSR-7レスポンスオブジェクトのデフォルトのステータスコードは`200` (OK)です。 `getStatusCode()`メソッドを使用して、PSR-7レスポンスオブジェクトのステータスコードを取得できます。

[statuscodes]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html

<figure markdown="1">
```php
$status = $response->getStatusCode();
```
<figcaption>Figure 3: Get response status code.</figcaption>
</figure>

PSR-7レスポンスオブジェクトをコピーして、次のような新しいステータスコードを割り当てることができます。

<figure markdown="1">
```php
$newResponse = $response->withStatus(302);
```
<figcaption>Figure 4: Create response with new status code.</figcaption>
</figure>

## The Response Headers

すべてのHTTPレスポンスにはヘッダーがあります。これらはHTTPレスポンスを記述するメタデータですが、レスポンスのボディには表示されません。
SlimのPSR-7レスポンスオブジェクトは、ヘッダーを検査および操作するためのいくつかのメソッドを提供します。

### Get All Headers

PSR-7レスポンスオブジェクトの`getHeaders()`メソッドを使用して、すべてのHTTPレスポンスヘッダーを連想配列として取得できます。
結果の連想配列のキーはヘッダー名であり、その値はそれぞれのヘッダー名の文字列値を持った数値配列です。

<figure markdown="1">
```php
$headers = $response->getHeaders();
foreach ($headers as $name => $values) {
    echo $name . ": " . implode(", ", $values);
}
```
<figcaption>Figure 5: Fetch and iterate all HTTP response headers as an associative array.</figcaption>
</figure>

### Get One Header

PSR-7レスポンスオブジェクトの`getHeader($name)`メソッドを使用して、単一のヘッダーの値を取得できます。
これは、指定されたヘッダー名の値の配列を返します。単一のHTTPヘッダーに複数の値がある場合もあるので注意してください！

<figure markdown="1">
```php
$headerValueArray = $response->getHeader('Vary');
```
<figcaption>Figure 6: Get values for a specific HTTP header.</figcaption>
</figure>

PSR-7レスポンスオブジェクトの`getHeaderLine($name)`メソッドを使用して、特定のヘッダーのすべての値を含むカンマ区切り文字列を取得することもできます。
`getHeader($name)`メソッドとは異なり、このメソッドはカンマ区切りの文字列を返します。

<figure markdown="1">
```php
$headerValueString = $response->getHeaderLine('Vary');
```
<figcaption>Figure 7: Get single header's values as comma-separated string.</figcaption>
</figure>

### Detect Header

PSR-7レスポンスオブジェクトの`hasHeader($name)`メソッドを使用して、ヘッダーの存在を確認できます。

<figure markdown="1">
```php
if ($response->hasHeader('Vary')) {
    // Do something
}
```
<figcaption>Figure 8: Detect presence of a specific HTTP header.</figcaption>
</figure>

### Set Header

ヘッダー値は、PSR-7レスポンスオブジェクトの`withHeader($name, $value)`メソッドで設定できます。

<figure markdown="1">
```php
$newResponse = $oldResponse->withHeader('Content-type', 'application/json');
```
<figcaption>Figure 9: Set HTTP header</figcaption>
</figure>

<div class="alert alert-info">
    <div><strong>注意</strong></div>
    <div>
    レスポンスオブジェクトは不変です。このメソッドは、新しいヘッダー値を持つResponseオブジェクトの<em>コピー</em>を返します。
    このメソッドは破壊的であり、同じヘッダー名で既に関連付けられている既存のヘッダー値を置き換えます。
    </div>
</div>

### Append Header

PSR-7レスポンスオブジェクトの`withAddedHeader($name, $value)`メソッドでヘッダー値を追加できます。

<figure markdown="1">
```php
$newResponse = $oldResponse->withAddedHeader('Allow', 'PUT');
```
<figcaption>Figure 10: Append HTTP header</figcaption>
</figure>

<div class="alert alert-info">
    <div><strong>注意</strong></div>
    <div>
    <code>withHeader()</code>メソッドとは異なり、このメソッドは同じヘッダー名に既に存在する値のセットに新しい値を追加します。
    レスポンスオブジェクトは不変です。このメソッドは、ヘッダー値が追加されたResponseオブジェクトのコピーを返却します。
    </div>
</div>

### Remove Header

Responseオブジェクトの `withoutHeader($name)` メソッドでヘッダーを削除できます。

<figure markdown="1">
```php
$newResponse = $oldResponse->withoutHeader('Allow');
```
<figcaption>Figure 11: Remove HTTP header</figcaption>
</figure>

<div class="alert alert-info">
    <div><strong>注意</strong></div>
    <div>
        レスポンスオブジェクトは不変です。このメソッドは、ヘッダー値が削除されたレスポンスオブジェクトのコピーを返します。
    </div>
</div>

## The Response Body

HTTPレスポンスには通常ボディがあります。 Slimは、最終的なHTTPレスポンスのボディを検査および操作できるPSR-7レスポンスオブジェクトを提供します。

PSR-7リクエストオブジェクトと同様に、PSR-7レスポンスオブジェクトは、`\Psr\Http\Message\StreamInterface`のインスタンスとしてボディを実装します。
PSR-7レスポンスオブジェクトの`getBody()`メソッドを使用して、HTTPレスポンスボディの`StreamInterface`インスタンスを取得できます。
`getBody()`メソッドは、送出されるHTTPレスポンスの長さが不明であるか、使用可能なメモリに対して大きすぎる場合に適しています。

<figure markdown="1">
```php
$body = $response->getBody();
```
<figcaption>Figure 12: Get HTTP response body</figcaption>
</figure>

処理結果の`\Psr\Http\Message\StreamInterface`インスタンスは、ベースとなるPHP`resource`の読み取り、反復、書き込みを行う次のメソッドを提供します。

* `getSize()`
* `tell()`
* `eof()`
* `isSeekable()`
* `seek()`
* `rewind()`
* `isWritable()`
* `write($string)`
* `isReadable()`
* `read($length)`
* `getContents()`
* `getMetadata($key = null)`

ほとんどの場合、PSR-7レスポンスオブジェクトに書き込む必要があります。次のような`write()`メソッドを使用して、`StreamInterface`インスタンスにコンテンツを書き込むことができます。

<figure markdown="1">
```php
$body = $response->getBody();
$body->write('Hello');
```
<figcaption>Figure 13: Write content to the HTTP response body</figcaption>
</figure>

PSR-7レスポンスオブジェクトのボディをまったく新しい`StreamInterface`インスタンスに置き換えることもできます。
これは、コンテンツをリモート先（ファイルシステムやリモートAPIなど）からHTTPレスポンスにパイプする場合でとくに便利です。
またPSR-7レスポンスオブジェクトのボディを`withBody(StreamInterface $body)`メソッドに置き換えることができます。
引数は`\Psr\Http\Message\StreamInterface`のインスタンスでなければなりません。

<figure markdown="1">
```php
$newStream = new \GuzzleHttp\Psr7\LazyOpenStream('/path/to/file', 'r');
$newResponse = $oldResponse->withBody($newStream);
```
<figcaption>Figure 14: Replace the HTTP response body</figcaption>
</figure>

<div class="alert alert-info">
    <div><strong>Reminder</strong></div>
    <div>
        レスポンスオブジェクトは不変です。このメソッドは、新しいボディを含むレスポンスオブジェクトのコピーを返します。
    </div>
</div>

## Returning JSON

Slimのレスポンスオブジェクトには、`withJson($data, $status, $encodingOptions)`を使用したカスタムメソッドがあり、JSONデータを返すプロセスを簡素化します。

`$data`パラメーターには、あなたが希望するデータ構造の返却用JSONが含まれます。`$status`はオプションであり、カスタムHTTPコードを返すために使用できます。
 `$encodingOptions`もオプションであり、 [`json_encode()`][json_encode]に使用されるのと同じエンコードオプションです。

単純なフォームでは、JSONデータはデフォルトの200 HTTPステータスコードで返されます。

<figure markdown="1">
```php
$data = array('name' => 'Bob', 'age' => 40);
$newResponse = $oldResponse->withJson($data);
```
<figcaption>Figure 15: Returning JSON with a 200 HTTP status code.</figcaption>
</figure>

カスタムHTTPステータスコードでJSONデータを返すこともできます。

<figure markdown="1">
```php
$data = array('name' => 'Rob', 'age' => 40);
$newResponse = $oldResponse->withJson($data, 201);
```
<figcaption>Figure 16: Returning JSON with a 201 HTTP status code.</figcaption>
</figure>

レスポンスの`Content-Type`は、`application/json;charset=utf-8`に自動的に設定されます。

データをJSONにエンコードする際、問題がある場合は[`json_last_error_msg()`][json_last_error_msg]の値を`$message`として、[`json_last_error()`][json_last_error]を`$code`として含む `\RuntimeException($message, $code)`がスローされます。

<div class="alert alert-info">
    <div><strong>Reminder</strong></div>
    <div>
        レスポンスオブジェクトは不変です。このメソッドは、新しいContent-Typeヘッダーを持つレスポンスオブジェクトのコピーを返します。
        このメソッドは破壊的であり、既存のContent-Typeヘッダーを置き換えます。
        また、<code>withJson()</code>が呼び出されたときに$statusが渡された場合、Statusも置き換えられます。
    </div>
</div>

[json_encode]: http://php.net/manual/en/function.json-encode.php
[json_last_error]: http://php.net/manual/en/function.json-last-error.php
[json_last_error_msg]: http://php.net/manual/en/function.json-last-error-msg.php

## Returning a Redirect

Slimのレスポンスオブジェクトには、リダイレクトを別のURLに返したい場合のために、 `withRedirect($url, $status = null)`のカスタムメソッドがあります。
オプションの`$status`コードとともに、クライアントをリダイレクトする場所の`$url`を指定します。
指定されていない場合のステータスコードは、デフォルトの`302`です。

<figure markdown="1">
```php
return $response->withRedirect('/new-url', 301);
```
<figcaption>Figure 17: Returning a redirect with an optional status code.</figcaption>
</figure>
