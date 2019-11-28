---
title: Templates
---

Slimには、従来のMVCフレームワークのようなビューレイヤーがありません。代わりに、Slimの"ビュー"はHTTPレスポンスです。
各Slimアプリケーションルートは、適切なPSR-7レスポンスオブジェクトの準備と、それを返す役割を果たします。

> Slimの"ビュー"はHTTPレスポンス.

そうは言っても、SlimプロジェクトはテンプレートをPSR7 Responseオブジェクトをレンダリングするのに役立つ
[Twig-View](#the-slimtwig-view-component)および[PHP-View](#the-slimphp-view-component)コンポーネントを提供します。

## The slim/twig-view component

[Twig-View][twigview]PHPコンポーネントは、アプリケーションで[Twig][twig]テンプレートをレンダリングするのに役立ちます。
このコンポーネントはPackagistで利用でき、Composerを使用すると次のように簡単にインストールできます。

[twigview]: https://github.com/slimphp/Twig-View
[twig]: http://twig.sensiolabs.org/

<figure markdown="1">
```
composer require slim/twig-view
```
<figcaption>Figure 1: Install slim/twig-view component.</figcaption>
</figure>

次に、下記のようにSlimアプリのコンテナーにサービスとしてコンポーネントを登録する必要があります。

<figure markdown="1">
```php
<?php
// アプリケーション作成
$app = new \Slim\App();

// コンテナー取得
$container = $app->getContainer();

// コンテナーをコンポーネントに登録
$container['view'] = function ($container) {
    $view = new \Slim\Views\Twig('path/to/templates', [
        'cache' => 'path/to/cache'
    ]);

    // Slimの拡張機能のインスタンス化と追加
    $router = $container->get('router');
    $uri = \Slim\Http\Uri::createFromEnvironment(new \Slim\Http\Environment($_SERVER));
    $view->addExtension(new \Slim\Views\TwigExtension($router, $uri));

    return $view;
};
```
<figcaption>Figure 2: Register slim/twig-view component with container.</figcaption>
</figure>

注："cache"をfalseに設定して無効にすることができます。開発環境で役立つ"auto_reload"オプションも参照してください。
詳細は[Twig environment options](http://twig.sensiolabs.org/doc/2.x/api.html#environment-options)で参照できます。

これでアプリルート内で`slim/twig-view`コンポーネントサービスを使用して、次のようにテンプレートをレンダリングし、PSR-7 Responseオブジェクトに書き込むことができます。

<figure markdown="1">
```php
// Render Twig template in route
$app->get('/hello/{name}', function ($request, $response, $args) {
    return $this->view->render($response, 'profile.html', [
        'name' => $args['name']
    ]);
})->setName('profile');

// Run app
$app->run();
```
<figcaption>Figure 3: Render template with slim/twig-view container service.</figcaption>
</figure>

この例では、ルートコールバック内で呼び出される$`$this->view`は、`view`コンテナサービスによって返される`\Slim\Views\Twig`インスタンスへの参照です。
`\Slim\Views\Twig`インスタンスの`render()`メソッドは、PSR-7 Responseオブジェクトを最初の引数として、Twigテンプレートパスを2番目の引数として、テンプレート変数の配列を最後の引数として受け入れます。`render()`メソッドは、ボディがたTwigテンプレートにレンダリングされた新しいPSR-7 Responseオブジェクトを返します。

### The path_for() method

`slim/twig-view`コンポーネントは、カスタムの`path_for()`メソッドをTwigテンプレートに公開します。
この関数を使用して、Slimアプリケーションの任意の名前付きルートへのURLを生成できます。`path_for()`メソッドは2つの引数を受け入れます。

1. ルート名
2. ルートプレースホルダー名とその置換値のハッシュ値

2番目の引数のキーは、選択されたルートのパターンプレースホルダーに対応する必要があります。
これは、上記のSlimアプリケーションの例に示されている"profile"という名前のルートのリンクURLを描画するTwigテンプレートの例です。

```html
{% raw %}
{% extends "layout.html" %}

{% block body %}
<h1>User List</h1>
<ul>
    <li><a href="{{ path_for('profile', { 'name': 'josh' }) }}">Josh</a></li>
</ul>
{% endblock %}
{% endraw %}
```

### Extending twig
Twigは、追加フィルター、メソッド、グローバル変数、タグなどで拡張できます。

フィルターを登録するには、コンテナーにビューコンポーネントを登録した後に以下を追加します。

<figure markdown="1">
```php
$filter = new Twig_SimpleFilter('rot13', function ($string) {
  return str_rot13($string);
});

$container->get('view')->getEnvironment()->addFilter($filter);
```
<figcaption>Figure 4: Registering a filter with Twig</figcaption>
</figure>

これは、twigに"rot13"フィルターを追加します

```html
{% raw %}
{# outputs "Slim Framework" #}
{{ 'Fyvz Senzrjbex'|rot13}}
{% endraw %}
```

メソッドを登録するには、コンテナーにビューコンポーネントを登録した後に次を追加します。

<figure markdown="1">
```php
$function = new Twig_SimpleFunction('shortest', function ($a, $b) {
  return strlen($a) <= strlen($b) ? $a : $b;
});

$container->get('view')->getEnvironment()->addFunction($function);
```
<figcaption>Figure 5: Registering a function with Twig</figcaption>
</figure>

これは、twigに"shortest"メソッドを追加します

```html
{% raw %}
{# outputs "Slim" #}
{{ shortest('Slim', 'Framework') }}
{% endraw %}
```

[twig documentation](https://twig.symfony.com/doc/2.x/advanced.html#creating-an-extension)で、Twig拡張の詳細を参照できます。

## The slim/php-view component

[PHP-View][phpview]PHPコンポーネントは、PHPテンプレートのレンダリングに役立ちます。
このコンポーネントはPackagistで利用可能で、次のようにComposerを使用してインストールできます。

[phpview]: https://github.com/slimphp/PHP-View

<figure markdown="1">
```
composer require slim/php-view
```
<figcaption>Figure 6: Install slim/php-view component.</figcaption>
</figure>

このコンポーネントをSlimアプリのコンテナーにサービスとして登録するには、次のようにします。

<figure markdown="1">
```php
<?php
// Create app
$app = new \Slim\App();

// Get container
$container = $app->getContainer();

// Register component on container
$container['view'] = function ($container) {
    return new \Slim\Views\PhpRenderer('path/to/templates/with/trailing/slash/');
};
```
<figcaption>Figure 7: Register slim/php-view component with container.</figcaption>
</figure>

ビューコンポーネントを使用して、次のようにPHPビューをレンダリングします。

<figure markdown="1">
```php

// Render PHP template in route
$app->get('/hello/{name}', function ($request, $response, $args) {
    return $this->view->render($response, 'profile.html', [
        'name' => $args['name']
    ]);
})->setName('profile');

// Run app
$app->run();
```
<figcaption>Figure 8: Render template with slim/php-view container service.</figcaption>
</figure>

## Other template systems

`Twig-View`および`PHP-View`コンポーネントに制限されません。
最終的にレンダリングされたテンプレート出力をPSR-7 Responseオブジェクトのボディに書き込むことを条件に、任意のPHPテンプレートシステムを使用できます。
