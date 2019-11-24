---
title: Request
---

Slimアプリのルートとミドルウェアには、Webサーバーが受信したHTTPリクエストを表すPSR-7リクエストオブジェクトが与えられます。
リクエストオブジェクトは、HTTPリクエストメソッド、ヘッダー、およびボディを検査および操作できる [PSR-7 ServerRequestInterface][psr7] を実装します。

[psr7]: http://www.php-fig.org/psr/psr-7/#3-2-1-psr-http-message-serverrequestinterface

## How to get the Request object

PSR-7リクエストオブジェクトは、次のようにルートコールバックの最初の引数としてSlimアプリケーションルートに注入されます。

<figure markdown="1">
```php
<?php
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

$app = new \Slim\App;
$app->get('/foo', function (ServerRequestInterface $request, ResponseInterface $response) {
    // Use the PSR-7 $request object

    return $response;
});
$app->run();
```
<figcaption>Figure 1: Inject PSR-7 request into application route callback.</figcaption>
</figure>

PSR-7リクエストオブジェクトは、次のように`middleware callble`の最初の引数としてSlimアプリケーションミドルウェアに注入されます。

<figure markdown="1">
```php
<?php
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

$app = new \Slim\App;
$app->add(function (ServerRequestInterface $request, ResponseInterface $response, callable $next) {
    // Use the PSR-7 $request object

    return $next($request, $response);
});
// Define app routes...
$app->run();
```
<figcaption>Figure 2: Inject PSR-7 request into application middleware.</figcaption>
</figure>

## The Request Method

すべてのHTTPリクエストには、通常次のいずれかのメソッドがあります。

* GET
* POST
* PUT
* DELETE
* HEAD
* PATCH
* OPTIONS

`getMethod()`というRequestオブジェクトメソッドを使用して、HTTPリクエストのメソッドを検査できます。

```php
$method = $request->getMethod();
```

これは一般的なタスクであるため、SlimのビルトインPSR-7実装は、trueまたはfalseを返すこれらの独自のメソッドも提供します。

* `$request->isGet()`
* `$request->isPost()`
* `$request->isPut()`
* `$request->isDelete()`
* `$request->isHead()`
* `$request->isPatch()`
* `$request->isOptions()`

HTTPリクエストメソッドのフェイクを作成、またはメソッドのオーバーライドができます。
これは、たとえば、 `GET`または`POST`リクエストのみをサポートする従来のWebブラウザーを使用して`PUT`リクエストを模倣する必要がある場合に役立ちます。

HTTPリクエストメソッドをオーバーライドするには、2つの方法があります。`POST`リクエストの本文に`_METHOD`パラメーターを含めることができます。 
HTTPリクエストでは、`application/x-www-form-urlencoded` コンテンツタイプを使用する必要があります。

<figure markdown="1">
```
POST /path HTTP/1.1
Host: example.com
Content-type: application/x-www-form-urlencoded
Content-length: 22

data=value&_METHOD=PUT
```
<figcaption>Figure 3: Override HTTP method with _METHOD parameter.</figcaption>
</figure>

カスタム`X-Http-Method-Override`HTTPリクエストヘッダーでHTTPリクエストメソッドをオーバーライドすることもできます。
これは、すべてのHTTPリクエストコンテンツタイプで機能します。

<figure markdown="1">
```
POST /path HTTP/1.1
Host: example.com
Content-type: application/json
Content-length: 16
X-Http-Method-Override: PUT

{"data":"value"}
```
<figcaption>Figure 4: Override HTTP method with X-Http-Method-Override header.</figcaption>
</figure>

`getOriginalMethod()`というPSR-7 Requestオブジェクトのメソッドを使用して、元の（オーバーライドされていない）HTTPメソッドを取得できます。

## The Request URI

すべてのHTTPリクエストには、要求されたアプリケーションリソースを識別するURIがあります。 HTTPリクエストURIはいくつかの要素を持ちます。

* Scheme (e.g. `http` or `https`)
* Host (e.g. `example.com`)
* Port (e.g. `80` or `443`)
* Path (e.g. `/users/1`)
* Query string (e.g. `sort=created&dir=asc`)

PSR-7リクエストオブジェクトの [URI object][psr7_uri] を`getUri()`メソッドで取得できます：

[psr7_uri]: http://www.php-fig.org/psr/psr-7/#3-5-psr-http-message-uriinterface

```php
$uri = $request->getUri();
```

PSR-7リクエストオブジェクトのURIは、それ自体がHTTPリクエストのURL部分を検査する次のメソッドを提供するオブジェクトです。

* `getScheme()`
* `getAuthority()`
* `getUserInfo()`
* `getHost()`
* `getPort()`
* `getPath()`
* `getBasePath()`
* `getQuery()` <small>(returns the full query string, e.g. `a=1&b=2`)</small>
* `getFragment()`
* `getBaseUrl()`

`getQueryParams()`を使用して、Requestオブジェクトの連想配列としてクエリパラメータを取得できます。

 `getQueryParam($key, $default = null)`を使用して、パラメーターが欠落している場合にオプションのデフォルト値を使用して、単一のクエリパラメーター値を取得することもできます。

<div class="alert alert-info">
    <div><strong>Base Path</strong></div>
    Slimアプリケーションのフロントコントローラーがドキュメントルートディレクトリ配下の物理サブディレクトリにある場合、Uriオブジェクトの<code>getBasePath()</code>メソッドを使用して、HTTPリクエストの（ドキュメントルートに対する）物理ベースパスを取得できます。 Slimアプリケーションがドキュメントルートの最上位ディレクトリにインストールされている場合、これは空の文字列になります。
</div>

## The Request Headers

すべてのHTTPリクエストにはヘッダーがあります。これらはHTTPリクエストを記述するメタデータですが、リクエストのボディには表示されません。 SlimのPSR-7 Requestオブジェクトは、ヘッダーを検査するためのいくつかのメソッドを提供します。

### Get All Headers

PSR-7 Requestオブジェクトの `getHeaders()`メソッドを使用して、すべてのHTTPリクエストヘッダーを連想配列として取得できます。
結果の連想配列のキーはヘッダー名であり、それらの値は、ヘッダー名の文字列型の値が数値型の配列として設定されています。

<figure markdown="1">
```php
$headers = $request->getHeaders();
foreach ($headers as $name => $values) {
    echo $name . ": " . implode(", ", $values);
}
```
<figcaption>Figure 5: Fetch and iterate all HTTP request headers as an associative array.</figcaption>
</figure>

### Get One Header

PSR-7 Requestオブジェクトの `getHeader($name)` メソッドを使用して、単一のヘッダーの値を取得できます。これは、指定されたヘッダー名の値の配列を返します。
単一のHTTPヘッダーに複数の値がある場合もあることに注意してください。

<figure markdown="1">
```php
$headerValueArray = $request->getHeader('Accept');
```
<figcaption>Figure 6: Get values for a specific HTTP header.</figcaption>
</figure>

また、PSR-7Requestオブジェクトの`getHeaderLine($name)`メソッドを使用して、特定のヘッダーのすべての値を含むコンマ区切りの文字列を取得することもできます。
`getHeader($name)`メソッドとは異なり、このメソッドはコンマ区切りの文字列を返します。

<figure markdown="1">
```php
$headerValueString = $request->getHeaderLine('Accept');
```
<figcaption>Figure 7: Get single header's values as comma-separated string.</figcaption>
</figure>

### Detect Header

PSR-7Requestオブジェクトの`hasHeader($name)`メソッドを使用して、ヘッダーの存在を確認できます。

<figure markdown="1">
```php
if ($request->hasHeader('Accept')) {
    // Do something
}
```
<figcaption>Figure 8: Detect presence of a specific HTTP request header.</figcaption>
</figure>

## The Request Body

すべてのHTTPリクエストにはボディがあります。 JSONまたはXMLデータを受け入れるSlimアプリケーションを構築している場合、PSR-7リクエストオブジェクトの`getParsedBody()`メソッドを使用して、
HTTPリクエストのボティ値をネイティブPHP形式に解析できます。 Slimは、JSON、XML、およびURLエンコードされたデータをすぐに解析できます。

<figure markdown="1">
```php
$parsedBody = $request->getParsedBody();
```
<figcaption>Figure 9: Parse HTTP request body into native PHP format</figcaption>
</figure>

* JSONリクエストは、`json_decode($input, true)を使用して連想配列に変換されます。
* XMLリクエストは、`simplexml_load_string($input)`を使用して`SimpleXMLElement`に変換されます。
* URLエンコードされたリクエストは、`parse_str($input)`を使用してPHP配列に変換されます。

URLエンコードされたリクエストでパラメーターが欠落している場合、オプションのデフォルト値である`getParsedBodyParam($key, $default = null)`を使用して、単一のパラメーター値を取得することもできます。

技術的に言えば、SlimのPSR-7リクエストオブジェクトは、HTTPリクエストのボディを`\Psr\Http\Message\StreamInterface`のインスタンスとして表します。 
PSR-7リクエストオブジェクトの`getBody()`メソッドを使用して、HTTPリクエストボディの`StreamInterface`インスタンスを取得できます。
受信したHTTPリクエストのサイズが不明であるか、使用可能なメモリに対して大きすぎる場合は、`getBody()`メソッドを勧めします。

<figure markdown="1">
```php
$body = $request->getBody();
```
<figcaption>Figure 10: Get HTTP request body</figcaption>
</figure>

取得結果の`\Psr\Http\Message\StreamInterface`インスタンスは、ベースとなるPHPリソースの読み取り、反復を行える次のメソッドを提供します。

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

### Reparsing the body

Requestオブジェクトで`getParsedBody`を複数回呼び出した場合に、その間にRequestボディが変更されても、ボディは一度しか解析されません。

ボディが確実に再解析されるようにするには、Requestオブジェクトのメソッドの`reparseBody`を使用できます。

## Uploaded Files

`$_FILES`のファイルアップロードは、Requestオブジェクトの`getUploadedFiles()`メソッドから利用できます。これは、`<input>` 要素の名前をキーとする配列を返します。

<figure markdown="1">
```php
$files = $request->getUploadedFiles();
```
<figcaption>Figure 11: Get uploaded files</figcaption>
</figure>

`$files`配列の各オブジェクトは、`\Psr\Http\Message\UploadedFileInterface`のインスタンスであり、次のメソッドをサポートしています。

* `getStream()`
* `moveTo($targetPath)`
* `getSize()`
* `getError()`
* `getClientFilename()`
* `getClientMediaType()`

POSTフォームを使用してファイルをアップロードする方法については、[cookbook](/docs/v3/cookbook/uploading-files.html)を参照してください。

## Request Helpers

SlimのPSR-7リクエストの実装には、以下に紹介する追加の独自のメソッドを提供され、HTTPリクエストをさらに調査するのに役立ちます。

### Detect XHR requests

Requestオブジェクトの`isXhr()`メソッドでXHRリクエストを検出できます。このメソッドは、`X-Requested-With`HTTPリクエストヘッダーの存在を検出し、その値が`XMLHttpRequest`であることを確認します。
※　XMLHttpRequest (XHR) は、JavaScriptなどのウェブブラウザ搭載のスクリプト言語でサーバとのHTTP通信を行うための、組み込みオブジェクト（API）。

<figure markdown="1">
```
POST /path HTTP/1.1
Host: example.com
Content-type: application/x-www-form-urlencoded
Content-length: 7
X-Requested-With: XMLHttpRequest

foo=bar
```
<figcaption>Figure 13: Example XHR request.</figcaption>
</figure>

```php
if ($request->isXhr()) {
    // Do something
}
```

### Content Type

HTTPリクエストのコンテンツタイプは、Requestオブジェクトの`getContentType()`メソッドで取得できます。
これは、HTTPクライアントによって提供される`Content-Type`ヘッダーの完全な値を返します。

```php
$contentType = $request->getContentType();
```

### Media Type

完全な`Content-Type`ヘッダーは必要ない場合があります。たとえばメディアタイプのみ必要な場合などです。
その場合はRequestオブジェクトの`getMediaType()`メソッドを使用して、HTTPリクエストのメディアタイプを取得できます。

```php
$mediaType = $request->getMediaType();
```

Requestオブジェクトの`getMediaTypeParams()`メソッドを使用して、追加されたメディアタイプパラメーターを連想配列として取得できます。

```php
$mediaParams = $request->getMediaTypeParams();
```

### Character Set

最も一般的なメディアタイプパラメータの1つは、HTTPリクエストの`character set`です。 Requestオブジェクトは、このメディアタイプパラメーターを取得する専用のメソッドを提供します。

```php
$charset = $request->getContentCharset();
```

### Content Length

HTTPリクエストコンテンツの長さは、Requestオブジェクトの`getContentLength()`メソッドで取得できます。

```php
$length = $request->getContentLength();
```

### Request Parameter

単一のリクエストパラメーター値を取得するには次のメソッドがあります。`getParam()`, `getQueryParam()`, `getParsedBodyParam()`, `getCookieParam()`, `getServerParam()`。
これらはPSR-7標準メソッドの`get*Params()`メソッドと同等のものとして提供されています。

たとえば、単一のサーバーパラメーターを取得するには

```php
$foo = $request->getServerParam('HTTP_NOT_EXIST', 'default_value_here');
```

## Route Object

ミドルウェアでは、ルートのパラメーターが必要になる場合があります。

この例では、まずユーザーがログインしていることを確認し、次にユーザーが表示しようとしている特定のビデオを表示する権限を持っていることを確認しています。

```php
    $app->get('/course/{id}', Video::class.":watch")->add(Permission::class)->add(Auth::class);

    //.. In the Permission Class's Invoke
    /** @var $route \Slim\Route */
    $route = $request->getAttribute('route');
    $courseId = $route->getArgument('id');
```

## Media Type Parsers

Slimはリクエストのメディアタイプを認識した場合、``$request->getParsedBody()``を介して利用可能な構造化データに解析します。これは通常配列ですが、XMLメディアタイプのオブジェクトです。

次のメディアタイプを認識して解析します。

* application/x-www-form-urlencoded
* application/json
* application/xml & text/xml

Slimに異なるメディアタイプのコンテンツを解析させたい場合は、素のボディを自分で解析するか、新しいメディアパーサーを登録する必要があります。
メディアパーサーは、 ``$input``文字列を受け取り、解析されたオブジェクトまたは配列を返すシンプルな呼び出しオブジェクトです。

新しいメディアパーサーをアプリケーションに登録するか、ミドルウェアをルーティングします。解析されたボディへはじめてアクセスする前に、パーサーを登録する必要があることに注意してください。
たとえば、``text/javascript``コンテンツタイプで送信されるJSONを自動的に解析するには、次のようにミドルウェアにメディアタイプパーサーを登録します。

```php
// Add the middleware
$app->add(function ($request, $response, $next) {
    // add media parser
    $request->registerMediaTypeParser(
        "text/javascript",
        function ($input) {
            return json_decode($input, true);
        }
    );

    return $next($request, $response);
});
```

## Attributes

PSR-7では、追加処理をするために、`objects/values`をリクエストオブジェクトに挿入できます。
アプリケーションミドルウェアはルートクロージャーに情報を渡す必要がある場合も多く、その方法は属性を介してリクエストオブジェクトに追加することです。

例）リクエストオブジェクトに値を設定する。

```php
$app->add(function ($request, $response, $next) {
    $request = $request->withAttribute('session', $_SESSION); //add the session storage to your request as [READ-ONLY]
    return $next($request, $response);
});
```

例）値を取得する方法

```php
$app->get('/test', function ($request, $response, $args) {
    $session = $request->getAttribute('session'); //get the session from the request

    return $response->write('Yay, ' . $session['name']);
});
```

リクエストオブジェクトにはバルク機能もあります。 `$request->getAttributes()`、 `$request->withAttributes()`
