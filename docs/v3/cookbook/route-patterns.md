---
title: Trailing / in route patterns
---

Slimは、末尾のスラッシュがあるURLパターンを、ないパターンとでは異なるものとして扱います。つまり、`/user`と`/user/`は異なるため、異なるコールバックを適用できます。

GETリクエストの場合、永続的なリダイレクトは問題ありませんが、POSTやPUTなどの他のリクエストメソッドの場合、ブラウザはGETメソッドを使用してそれらのリクエストを送信します。これを回避するには、末尾のスラッシュを削除して、独自に修正したURLを次のミドルウェアに渡すだけです。

末尾が`/`で終わるURLと`/`がないURLとで同様にリダイレクト/リライトしたい場合は、このミドルウェアを追加できます。

If you want to redirect/rewrite all URLs that end in a `/` to the non-trailing `/` equivalent, then you can add this middleware:

```php
use Psr\Http\Message\RequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

$app->add(function (Request $request, Response $response, callable $next) {
    $uri = $request->getUri();
    $path = $uri->getPath();
    if ($path != '/' && substr($path, -1) == '/') {
        // permanently redirect paths with a trailing slash
        // to their non-trailing counterpart
        $uri = $uri->withPath(substr($path, 0, -1));
        
        if($request->getMethod() == 'GET') {
            return $response->withRedirect((string)$uri, 301);
        }
        else {
            return $next($request->withUri($uri), $response);
        }
    }

    return $next($request, $response);
});
```

または、[oscarotero/psr7-middlewares' TrailingSlash](//github.com/oscarotero/psr7-middlewares#trailingslash)ミドルウェアを検討してください。これにより、すべてのURLに末尾のスラッシュを強制的に追加することもできます。

```php
use Psr7Middlewares\Middleware\TrailingSlash;

$app->add(new TrailingSlash(true)); // trueで末尾にスラッシュを追加します (falseは逆に削除します)
```
