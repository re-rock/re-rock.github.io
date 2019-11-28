---
title: HTTP Caching
---

Slim3は、HTTPキャッシングにオプションのスタンドアロンPHPコンポーネントの[slimphp/Slim-HttpCache](https://github.com/slimphp/Slim-HttpCache)を使用します。
このコンポーネントを使用して、アプリケーションのアウトプットがクライアント側キャッシュに保持される時期と期間を制御する`Cache`、`Expires`、`ETag`、そして `Last-Modified`ヘッダーを含むレスポンスを作成して返すことができます。
支障なくこの機能を使用するには、php.iniの設定で"session.cache_limiter"を空の文字列で設定する必要があります。

## Installation

プロジェクトのルートディレクトリから次のbashコマンドを実行します。

```bash
composer require slim/http-cache
```

## Usage

`slimphp/Slim-HttpCache`コンポーネントには、サービスプロバイダーとアプリケーションミドルウェアが含まれています。次のように両方をアプリケーションに追加する必要があります。

```php
// Register service provider with the container
$container = new \Slim\Container;
$container['cache'] = function () {
    return new \Slim\HttpCache\CacheProvider();
};

// Add middleware to the application
$app = new \Slim\App($container);
$app->add(new \Slim\HttpCache\Cache('public', 86400));

// Create your application routes...

// Run application
$app->run();
```

## ETag

サービスプロバイダーの`withEtag()`メソッドを使用して、希望するの`ETag`ヘッダーを持つResponseオブジェクトを作成します。
このメソッドはPSR7レスポンスオブジェクトを受け入れ、新しいHTTPヘッダーを含むクローン化されたPSR7レスポンスを返します。

```php
$app->get('/foo', function ($req, $res, $args) {
    $resWithEtag = $this->cache->withEtag($res, 'abc');

    return $resWithEtag;
});
```

## Expires

サービスプロバイダーの`withExpires()`メソッドを使用して、希望するの`Expires`ヘッダーを持つResponseオブジェクトを作成します。
このメソッドはPSR7レスポンスオブジェクトを受け入れ、新しいHTTPヘッダーを含むクローン化されたPSR7レスポンスを返します。

```php
$app->get('/bar',function ($req, $res, $args) {
    $resWithExpires = $this->cache->withExpires($res, time() + 3600);

    return $resWithExpires;
});
```

## Last-Modified

サービスプロバイダーの`withLastModified()`メソッドを使用して、目的の`Last-Modified`ヘッダーを持つResponseオブジェクトを作成します。
このメソッドはPSR7レスポンスオブジェクトを受け入れ、新しいHTTPヘッダーを含むクローン化されたPSR7レスポンスを返します。

```php
//Example route with LastModified
$app->get('/foobar',function ($req, $res, $args) {
    $resWithLastMod = $this->cache->withLastModified($res, time() - 3600);

    return $resWithLastMod;
});
```
