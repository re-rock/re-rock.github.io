---
title: Application
---

アプリケーション（または `Slim\App`）は、Slimアプリケーションへのエントリポイントであり、コールバックまたはコントローラーにリンクするルートを登録するために使用されます。

```php
// instantiate the App object
$app = new \Slim\App();

// Add route callbacks
$app->get('/', function ($request, $response, $args) {
    return $response->withStatus(200)->write('Hello World!');
});

// Run application
$app->run();
```

## Application Configuration

アプリケーションは引数を1つだけ受け入れます。これは、自動的に作成されるデフォルトの[コンテナー](/docs/v3/concepts/di.html) を設定するコンテナインスタンスまたは配列のいずれかです。

Slimで使用される設定もいくつかあります。これらは、`settings`構成キーに保存されます。アプリケーション固有の設定を追加することもできます。

たとえば、Slim設定の`displayErrorDetails`をtrueに設定し、Monologを次のように構成することもできます。

```php
$config = [
    'settings' => [
        'displayErrorDetails' => true,

        'logger' => [
            'name' => 'slim-app',
            'level' => Monolog\Logger::DEBUG,
            'path' => __DIR__ . '/../logs/app.log',
        ],
    ],
];
$app = new \Slim\App($config);
```


### Retrieving Settings

設定はDIコンテナに保存されるため、コンテナファクトリの設定キーを使用してアクセスできます。

```php
$loggerSettings = $container->get('settings')['logger'];
```

また、`$this`を介してルートcallableでそれらにアクセスできます

```php
$app->get('/', function ($request, $response, $args) {
    $loggerSettings = $this->get('settings')['logger'];
    // ...
});
```

### Updating Settings

コンテナの初期化後にDIコンテナに保存されている設定を追加または更新する必要がある場合は、設定コンテナで`replace`メソッドを使用できます。

```php
$settings = $container->get('settings');
$settings->replace([
    'displayErrorDetails' => true,
    'determineRouteBeforeAppMiddleware' => true,
]);
```

## Slim Default Settings

Slimには上書き可能な以下のデフォルト設定があります。

<dl>
<dt><code>httpVersion</code></dt>
Responseオブジェクトが使用するプロトコルバージョン。

    <dd><a href="/docs/v3/objects/response.html">レスポンス</a>オブジェクトが使用するプロトコルバージョン
        <br>(Default: <code>'1.1'</code>)</dd>
<dt><code>responseChunkSize</code></dt>
    <dd>ブラウザーに送信したときのレスポンスボディから読み取られる各チャンクのサイズ。
        <br>(Default: <code>4096</code>)</dd>
<dt><code>outputBuffering</code></dt>
<code>false</code>の場合、出力バッファリングは有効になりません。
    <dd><code>'append'</code>または<code>'prepend'</code>の場合、echoまたはprintステートメントがキャプチャされ、ルートcallableから返されたレスポンスに追加または追加されます。
        <br>(Default: <code>'append'</code>)</dd>
<dt><code>determineRouteBeforeAppMiddleware</code></dt>
    <dd>trueの場合、ミドルウェアが実行される前にルートが計算されます。これは、必要に応じてミドルウェアでルートパラメーターを検査できることを意味します。
    <br>(Default: <code>false</code>)</dd>
<dt><code>displayErrorDetails</code></dt>
    <dd>trueの場合、例外に関する追加情報が<a href="/docs/v3/handlers/error.html">デフォルトのエラーハンドラー</a>によって表示されます。
    <br>(Default: <code>false</code>)</dd>
<dt><code>addContentLengthHeader</code></dt>
<dd>trueの場合、Slimは<code>Content-Length</code> ヘッダーを応答に追加します。 New Relicなどのランタイム分析ツールを使用している場合は、これを無効にする必要があります。
    <br>(Default: <code>true</code>)</dd>
<dt><code>routerCacheFile</code></dt>
<dd>FastRouteルートをキャッシュするためのファイル名。書き込み可能なディレクトリ内の有効なファイル名に設定する必要があります。ファイルが存在しない場合は、最初の実行時に正しいキャッシュ情報で作成されます。<br> <code>false</code>に設定すると、FastRouteキャッシュシステムが無効になります。
    <br>(Default: <code>false</code>)</dd>
</dl>
