---
title: System Error Handler
---

物事がうまくいかない。エラーを予言することはできませんが、予測することはできます。各Slimフレームワークアプリケーションには、キャッチされなかったすべてのPHPの例外を受け取るエラーハンドラーがあります。このエラーハンドラーは、現在のHTTPリクエストおよびレスポンスオブジェクトも受け取ります。エラーハンドラーは、HTTPクライアントに返される適切なResponseオブジェクトを準備し、そして返す必要があります。

## Default error handler

デフォルトのエラーハンドラーは非常に基本的なものです。レスポンスステータスコードを`500`に設定し、レスポンスコンテンツタイプを `text/html`に設定し、一般的なエラーメッセージをレスポンスボディに追加します。

これはおそらく実際に運用するアプリケーションには適していません。独自のスリムアプリケーションエラーハンドラーを実装することを強くオススメします。

デフォルトのエラーハンドラーには、詳細な診断エラー情報を含めることもできます。これを有効にするには、`displayErrorDetails`設定をtrueに設定する必要があります。

```php
$configuration = [
    'settings' => [
        'displayErrorDetails' => true,
    ],
];
$c = new \Slim\Container($configuration);
$app = new \Slim\App($c);
```

## Custom error handler

Slim FrameworkアプリケーションのエラーハンドラーはPimpleサービスです。アプリケーションコンテナーでカスタムPimpleファクトリメソッドを定義することにより、独自のエラーハンドラーに置き換えることができます。

ハンドラーを注入する方法は2つあります。

### Pre App

```php
$c = new \Slim\Container();
$c['errorHandler'] = function ($c) {
    return function ($request, $response, $exception) use ($c) {
        return $response->withStatus(500)
            ->withHeader('Content-Type', 'text/html')
            ->write('Something went wrong!');
    };
};
$app = new \Slim\App($c);
```

### Post App

```php
$app = new \Slim\App();
$c = $app->getContainer();
$c['errorHandler'] = function ($c) {
    return function ($request, $response, $exception) use ($c) {
        return $response->withStatus(500)
            ->withHeader('Content-Type', 'text/html')
            ->write('Something went wrong!');
    };
};
```

この例では、callbleを返す新しい`errorHandler`ファクトリーを定義します。返却されるcallableは3つの引数を受け入れます。

1. `\Psr\Http\Message\ServerRequestInterface`インスタンス
2. `\Psr\Http\Message\ResponseInterface`インスタンス
3. `\Exception`インスタンス

callableは、指定された例外に適した新しい`\Psr\Http\Message\ResponseInterface` インスタンスを返さなければなりません。

### Class-based error handler

エラーハンドラーは、呼び出し可能なクラスとして定義することもできます。

```php
class CustomHandler {
   public function __invoke($request, $response, $exception) {
        return $response
            ->withStatus(500)
            ->withHeader('Content-Type', 'text/html')
            ->write('Something went wrong!');
   }
}
```

そして次のように添付します

```php
$app = new \Slim\App();
$c = $app->getContainer();
$c['errorHandler'] = function ($c) {
    return new CustomHandler();
};
```

これにより、より洗練されたハンドラーを定義したり、組み込みの`Slim\Handlers\*`クラスを拡張/オーバーライドしたりできます。

### Handling other errors

注：次の4種類の例外は、カスタム`errorHandler`では処理されません。

- `Slim\Exception\MethodNotAllowedException`: これはカスタム[`notAllowedHandler`](/docs/v3/handlers/not-allowed.html)を介して処理できます。
- `Slim\Exception\NotFoundException`: これはカスタム[`notFoundHandler`](/docs/v3/handlers/not-found.html)を介して処理できます。
- Runtime PHP errors (PHP 7+ only): これはカスタム[`phpErrorHandler`](/docs/v3/handlers/php-error.html)を介して処理できます。
- `Slim\Exception\SlimException`: このタイプの例外はSlim内部にあり、その処理をオーバーライドすることはできません。

### Disabling

Slimのエラー処理を完全に無効にするには、コンテナーからエラーハンドラーを削除するだけです

```php
unset($app->getContainer()['errorHandler']);
unset($app->getContainer()['phpErrorHandler']);
```

これによってアプリケーションで発生した例外はSlimによって処理されなくなるため、代わりに例外処理行う責任が発生します。
