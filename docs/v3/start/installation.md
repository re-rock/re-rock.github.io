---
title: Installation
---

## システム要件

* URL書き換え可能なWebサーバ
* PHP 5.5以上

## インストール

Slimは[Composer](https://getcomposer.org/)からインストールすることをお勧めします。
プロジェクトのルートディレクトリに移動し、以下に示すbashコマンドを実行します。このコマンドは、
Slimフレームワークとそのサードパーティの依存関係ライブラリをプロジェクトの`vendor`ディレクトリ
にダウンロードします。

```bash
composer require slim/slim "^3.12"
```

あなたのプロジェクトにComposer autoloaderを含めたらSlimを始める準備が整いました。

```php
<?php
require 'vendor/autoload.php';
```

## Composerのインストール方法

Composerをまだインストールしていない場合、ダウンロードページの指示に従って簡単にできます。
