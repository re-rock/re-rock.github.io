---
title: Flash Messages
---

## Install

Composerでインストールします。

``` bash
$ composer require slim/flash
```

利用するにはSlim 3.0.0以上である必要があります。

## Usage

```php
// PHPセッション開始
session_start(); //デフォルトではセッションストレージが必要です

$app = new \Slim\App();

// DIコンテナーの取得
$container = $app->getContainer();

// プロバイダーを登録
$container['flash'] = function () {
    return new \Slim\Flash\Messages();
};

$app->get('/foo', function ($req, $res, $args) {
    // 次のリクエストのフラッシュメッセージを設定する
    $this->flash->addMessage('Test', 'This is a message');

    // リダイレクト
    return $res->withStatus(302)->withHeader('Location', '/bar');
});

$app->get('/bar', function ($req, $res, $args) {
    // 現在のリクエストで使用されるメッセージを追加します
    $this->flash->addMessageNow('Test', 'This is another message');

    // 前のリクエストからフラッシュメッセージを取得する
    $messages = $this->flash->getMessages();

    // 両方のフラッシュメッセージを返します
    print_r($messages);
});

$app->run();
```

メッセージは文字列、オブジェクト、または配列である可能性があることに注意してください。ストレージが処理できるものを確認してください。
