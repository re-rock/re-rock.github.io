---
title: Getting and Mocking the Environment
---

Environmentオブジェクトは、`$_SERVER`スーパーグローバル配列にカプセル化され、SlimアプリケーションをPHPグローバル環境から分離します。
SlimアプリケーションをPHPグローバル環境から切り離すことにより、グローバル環境に依頼（または依頼しない）HTTPリクエストを作成できます。
これは、ユニットテストおよびサブリクエストの開始に特に役立ちます。次のように、Slimアプリケーションのどこからでも現在のEnvironmentオブジェクトを取得できます。

```php
$container = $app->getContainer();
$environment = $container['environment'];
```

## Environment Properties

各Slimアプリケーションには、アプリケーションの動作を決定するさまざまなプロパティを持つEnvironmentオブジェクトがあります。
これらのプロパティの多くは、`$_SERVER`スーパーグローバル配列にあるプロパティを反映しています。プロパティには必須のものとオプションのものがあります。

### 必須のプロパティ

REQUEST_METHOD
:   HTTPリクエストメソッド。メソッドは次のものである必要があります。: "GET", "POST", "PUT", "DELETE", "HEAD", "PATCH", "OPTIONS"

SCRIPT_NAME
:   ドキュメントルートに対してのフロントコントローラーPHPスクリプトへの絶対パス名。Webサーバーによって実行されるURL書き換えを無視します。

REQUEST_URI
:   Webサーバーによって実行されるURL書き換えの変更を含む、HTTPリクエストURIの絶対パス名。

QUERY_STRING
:   HTTPリクエストのURIパス以降の部分。"?"は含まれません。現在のHTTPリクエストでクエリ文字列が指定されていない場合、空の文字列になる場合があります。

SERVER_NAME
:   現在のスクリプトが実行されているサーバーホストの名前。スクリプトが仮想ホストで実行されている場合、これはその仮想ホストに定義された値になります。

SERVER_PORT
:   Webサーバーが通信に使用しているサーバーマシンのポート。デフォルト設定の場合は'80'です。SSLを使用した場合、独自に定義済みのセキュアHTTPポートに変更されます。

HTTPS
:   スクリプトがHTTPSプロトコルを介して照会された場合、何かしらの値を設定します。

### Optional Properties

CONTENT_TYPE
:   HTTPリクエストのコンテンツタイプ（例：`application/json;charset=utf8`）

CONTENT_LENGTH
:   HTTPリクエストのコンテンツの長さ。これを設定する場合は整数でなければなりません。

HTTP_*
:   クライアントによって送信されたHTTPリクエストヘッダー。これらの値は、`$_SERVER`スーパーグローバル配列の対応する値と同じです。設定する場合、これらの値は "HTTP_" プレフィックスを保持する必要があります。

PHP_AUTH_USER
:   HTTP`Authentication`ヘッダーのデコードされたユーザー名。

PHP_AUTH_PW
:   HTTP`Authentication`ヘッダーのデコードされたパスワード。

PHP_AUTH_DIGEST
:   HTTPクライアントによって送信された素のHTTP`Authentication`ヘッダー。

AUTH_TYPE
:   HTTP`Authentication`ヘッダーの認証タイプ（例："Basic", "Digest"）。

slim.files
:   実装の配列`\Psr\Http\Message\UploadedFileInterface`（たとえば、ネイティブのSlimフレームワークでは`\Slim\Http\UploadedFile`）

## Mock Environments

各Slimアプリケーションは、グローバル環境からの情報を使用して、Environmentオブジェクトをインスタンス化します。
また、カスタム情報を使用してモック環境オブジェクトを作成することもできます。モックEnvironmentオブジェクトは、単体テストを作成するときにのみ役立ちます。

```php
$env = \Slim\Http\Environment::mock([
    'REQUEST_METHOD' => 'POST',
    'REQUEST_URI' => '/foo/bar',
    'QUERY_STRING' => 'abc=123&foo=bar',
    'SERVER_NAME' => 'example.com',
    'CONTENT_TYPE' => 'multipart/form-data',
    'slim.files' => [
        'field1' => new UploadedFile('/path/to/file1', 'filename1.txt', 'text/plain', filesize('/path/to/file1')),
        'field2' => new UploadedFile('/path/to/file2', 'filename2.txt', 'text/plain', filesize('/path/to/file2')),
    ],
]);
```
