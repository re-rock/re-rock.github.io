---
title: Deployment
---
おめでとうございます！ここに到達したということは、Slimを使用して素晴らしいものを作ることができたことになります。ただパーティーの時間にはまだ早いです。アプリケーションを運用サーバーにプッシュする必要があります。

デプロイには多くの方法があります。このセクションでは、さまざまなセットアップに関する注意事項を提供します。

## 本番環境でエラー表示を無効にする

最初に行うことは、設定（skeletonから作成した場合の`src/settings.php`）を修正し、完全なエラー詳細を公開しないようにすることです。

```php
  'displayErrorDetails' => false, // set to false in production
```

また、PHPインストールがエラーを表示しないように`php.ini`で設定されていることを確認してください。

```ini
display_errors = 0
```

## サーバにデプロイ

あなたがサーバを設定できる状況で次のような自動デプロイツールを使用する場合、デプロイメント・プロセスを設定する必要があります。

* Deploybot
* Capistrano
* Script controlled with Phing, Make, Ant, etc.

[Web Servers](/docs/v3/start/web-servers.html)のドキュメントを確認してWebサーバの設定を行ってください。


## 共有サーバにデプロイ

共有サーバーでApacheを実行している場合、Webサーバーのルートディレクトリ（通常は`htdocs`、`public`、`public_html`または`www`）に以下の内容の`.htaccess`ファイルを作成する必要があります。

```apache
<IfModule mod_rewrite.c>
   RewriteEngine on
   RewriteRule ^$ public/     [L]
   RewriteRule (.*) public/$1 [L]
</IfModule>
```

('public'をあなたのドメイン名に置き換えてください。例）example.com/$1)

次に、Slimプロジェクトを構成するすべてのファイルをWebサーバーにアップロードします。共有ホストの場合はおそらくFTP経由で行われると思いますが、FilezillaなどのFTPクライアントを使用してこれを行うことができます。
