---
title: CSRF Protection
---

Slim3は、オプションのスタンドアロンPHPコンポーネントである[slimphp/Slim-Csrf](https://github.com/slimphp/Slim-Csrf)を使用して、CSRF（クロスサイトリクエストフォージェリ）からアプリケーションを保護します。
このコンポーネントはクライアントサイドのHTMLフォームから送られる、POSTリクエストのシーケンスを検証する一意のトークンをリクエストごとに生成します。

## Installation

プロジェクトのルートディレクトリから次のbashコマンドを実行します。

```bash
composer require slim/csrf
```

## Usage

`slimphp/Slim-Csrf`コンポーネントには、アプリケーションミドルウェアが含まれています。次のようにアプリケーションに追加します。

```php
// Add middleware to the application
$app = new \Slim\App;
$app->add(new \Slim\Csrf\Guard);

// Create your application routes...

// Run application
$app->run();
```

## Fetch the CSRF token name and value

最新のCSRFトークンの名前と値は、PSR7リクエストオブジェクトの属性として利用できます。CSRFトークンの名前と値は、リクエストごとに一意です。
このようにCSRFトークンの名前と値を取得できます。

```php
$app->get('/foo', function ($req, $res, $args) {
    // CSRF token name and value
    $nameKey = $this->csrf->getTokenNameKey();
    $valueKey = $this->csrf->getTokenValueKey();
    $name = $req->getAttribute($nameKey);
    $value = $req->getAttribute($valueKey);

    // "/bar"にPOSTするHTMLフォームをレンダリングします。
    // これはnameとvalueの値を保持した2つのhidden属性のinputフィールドを持ちます。
    // <input type="hidden" name="<?= $nameKey ?>" value="<?= $name ?>">
    // <input type="hidden" name="<?= $valueKey ?>" value="<?= $value ?>">
});

$app->post('/bar', function ($req, $res, $args) {
    // ここまで到達すればCSRF保護は成功です。
});
```

CSRFトークンの名前と値をテンプレートに渡して、HTMLフォームのPOSTリクエストで送信できるようにする必要があります。
多くの場合、HTMLフォームのhiddenフィールドとして保存されます。

その他の使用例とドキュメントについては、[slimphp/Slim-Csrf](https://github.com/slimphp/Slim-Csrf)のページをご覧ください。
