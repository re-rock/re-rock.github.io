---
title: Web Servers
---

通常、フロントコントローラーパターンを使用してWebサーバが受信した適切なHTTPリクエストを単一の
PHPファイルにまとめます。通常、フロントコントローラーパターンを使用して、Webサーバーが受信した適切な
HTTPリクエストを単一のPHPファイルに転送します。
以下の手順では、URLに一致するファイルやディレクトリが存在しない場合、PHPフロントコントローラーファイルに
HTTPリクエストを送信するようにWebサーバーに指示する方法を説明します。

## PHPビルトインサーバ

ターミナルで次のコマンドを実行してlocalhost Webサーバを起動します。
サーバは`./public/`が`index.php`ファイルを持つパブリックアクセス可能なディレクトリであると想定してアクセスします。

```bash
php -S localhost:8888 -t public public/index.php
```

`index.php`をエントリポイントとして使用していない場合も適切に変更されます。

## Apacheの設定

`.htaccess`ファイルと`index.php`ファイルが同じパブリックアクセス可能なディレクトリにあることを確認してください。
`.htaccess`ファイルには次のコードが含まれている必要があります。

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```

この`.htaccess`ファイルには、URLの書き換えが必要です。 Apacheのhttpd.confで`mod_rewrite`モジュールを有効にし、
`VirtualHost`に`AllowOverride`オプションが設定されていることを確認してください。
これにより、`.htaccess`書き換えルールを使用できます。

```
AllowOverride All
```

## Nginxの設定

これは、`example.com`ドメインのNginバーチャルホスト設定の例です。ポート80でHTTP接続を受け付けています。
PHP-FPMサーバがポート9000で実行されていることを前提としています。
`server_name`、`error_log`、`access_log`および`root`ディレクティブをあなたの環境に合わせて設定してください。
`root`ディレクティブは、アプリケーションの公開ルートディレクトリへのパスです。Slimアプリの`index.php`フロントコントローラーファイルは
このディレクトリになければなりません。

```
server {
    listen 80;
    server_name example.com;
    index index.php;
    error_log /path/to/example.error.log;
    access_log /path/to/example.access.log;
    root /path/to/public;

    location / {
        try_files $uri /index.php$is_args$args;
    }

    location ~ \.php {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        fastcgi_index index.php;
        fastcgi_pass 127.0.0.1:9000;
    }
}
```

## HipHop Virtual Machine

HipHopバーチャルマシンの設定ファイルには、以下のコードが含まれている必要があります（その他必要な設定と一緒に）。
`SourceRoot`設定を変更して、Slimアプリのドキュメントルートディレクトリを指すようにしてください。
(HHVM : C++で実装されたPHP実行環境)

```
Server {
    SourceRoot = /path/to/public/directory
}

ServerVariables {
    SCRIPT_NAME = /index.php
}

VirtualHost {
    * {
        Pattern = .*
        RewriteRules {
                * {
                        pattern = ^(.*)$
                        to = index.php/$1
                        qsa = true
                }
        }
    }
}
```

## IIS

同じ公開ディレクトリに`Web.config`と`index.php` ファイルがあることを確認してください。
`Web.config`は以下のコードが含まれている必要があります。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule name="slim" patternSyntax="Wildcard">
                    <match url="*" />
                    <conditions>
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="index.php" />
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```

## lighttpd

lighttpdの設定ファイルには、以下のコードが含まれている必要があります（その他必要な設定と一緒に）。
またlighttpdのバージョンは`1.4.24.`以上である必要があります。

```
url.rewrite-if-not-file = ("(.*)" => "/index.php/$0")
```

以上からわかるように、Slimのindex.phpがプロジェクトのルートフォルダ(wwwルート)にあることが前提となります。
