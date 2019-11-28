---
title: Using Atlas 3 with Slim
---

このクックブックエントリでは、[Atlas 3](http://atlasphp.io) ORMとそのコマンドラインツールをSlimで使用する方法について説明します。

## Installation

Composerを使用してAtlasをインストールします。 ORMパッケージとCLIパッケージは別々に提供されます。これは、コマンドラインツールは開発時にのみ必要になる可能性が高いためです。

```
composer require atlas/orm ~3.0
composer require --dev atlas/cli ~2.0
```

> **Note:**
>
> PHPStormを使用している場合は、IDEメタファイルをプロジェクトにコピーして、
> Atlasクラスのフルオートコンプリートを取得できます。
>
> ```
> cp ./vendor/atlas/orm/resources/phpstorm.meta.php ./.phpstorm.meta.php
> ```

## Settings and Configuration

次に、いくつかのデータベース接続設定を構成に追加する必要があります。任意の配列キーに追加できます。次の例では、['settings'] ['atlas']を使用しています。
`['settings']['atlas']`:

```php
<?php
// ./config/settings.php
return [
    'settings' => [
        'displayErrorDetails' => true,
        'determineRouteBeforeAppMiddleware' => false,
        'atlas' => [
            'pdo' => [
                'mysql:dbname=testdb;host=localhost',
                'username',
                'password',
            ],
            'namespace' => 'DataSource',
            'directory' => dirname(__DIR__) . '/src/classes/DataSource',
        ]
    ]
];
```

`pdo`要素は、[PDO connection](https://secure.php.net/manual/en/pdo.construct.php)の引数として使用されます。

`namespace`と`directory`要素は、Atlasデータソースクラスの名前空間と、スケルトンジェネレーターによって保存されるディレクトリを指定します。
`composer.json`には、これらのPSRオートローダーエントリが必要です。上記の例は、Slim [tutorial application](https://github.com/slimphp/Tutorial-First-Application)に対応しています。

## Generating Skeleton Classes

これで、Atlas CLIツールを使用してスケルトンデータソースクラスを生成できます。
まず、ターゲットディレクトリを作成し、次にスケルトンコマンドを発行します。

```
mkdir -p ./src/classes/DataSource
php ./vendor/bin/atlas-skeleton.php ./config/settings.php settings.atlas
```

最初の引数の`atlas-skeleton`は、Slim設定ファイルへのパスです。 2番目の引数は、settings内のAtlas配列のキーとなるドット区切りリストです。

そのコマンドを使用して、Atlasはデータベースを調べ、スキーマ内の各テーブルの一連のクラスと特性を生成します。
各テーブルに対して生成されたファイルは[こちら](http://atlasphp.io/cassini/skeleton/usage.html#1-2-1-2)で確認できます。

## Container Service Definition

データマッパークラスを配置したら、Atlas ORMをスリムコンテナーのサービスとして追加できるようになります。

```php
<?php
$container = $app->getContainer();
$container['atlas'] = function ($container) {
    $args = $container['settings']['atlas']['pdo'];
    $atlasBuilder = new AtlasBuilder(...$args);
    $atlasBuilder->setFactory(function ($class) use ($container) {
        if ($container->has($class)) {
            return $container->get($class);
        }

        return new $class();
    });
    return $atlasBuilder->newAtlas();
};
```

これは、AtlasBuilderクラスを使用して、設定構成からのPDOコネクション引数を使用して、新しいORMインスタンスを作成して返します。

このサービス定義は、特定のクラス構成のファクトリとしてコンテナを使用するようにAtlasに設定します。
コンテナ内でAtlasのTableEventまたはMapperEventクラスのサービス定義がある場合、Atlasはコンテナを使用してそれらを構築します。

> **Note:**
>
> このサービス定義は、特定のクラス構成のファクトリとしてコンテナを使用するようにAtlasに設定します。
> コンテナ内でAtlasのTableEventまたはMapperEventクラスのサービス定義がある場合、Atlasはコンテナを使用してそれらを構築します。

## Using the ORM

この時点で、あなたの処理コードでAtlas ORMの全機能を使用できます。
次の例は、[tutorial application](https://github.com/slimphp/Tutorial-First-Application)からの抜粋です。

```php
<?php
use DataSource\Ticket\Ticket;
use DataSource\Component\Component;

$app->get('/tickets', function (Request $request, Response $response) {
    $this->logger->addInfo("Ticket list");
    $tickets = $this->atlas->select(Ticket::class)->fetchRecordSet();

    $response = $this->view->render(
        $response,
        "tickets.phtml",
        [
            "tickets" => $tickets,
            "router" => $this->router
        ]
    );
    return $response;
});

$app->get('/ticket/{id}', function (Request $request, Response $response, $args) {
    $ticket_id = (int) $args['id'];
    $ticket = $this->atlas->fetchRecord(Ticket::class, $ticket_id);

    $response = $this->view->render(
        $response,
        "ticketdetail.phtml",
        [
            "ticket" => $ticket
        ]
    );
    return $response;
})->setName('ticket-detail');

$app->post('/ticket/new', function (Request $request, Response $response) {
    $data = $request->getParsedBody();

    $ticket = $this->atlas->newRecord(Ticket::class);
    $ticket->title = filter_var($data['title'], FILTER_SANITIZE_STRING);
    $ticket->description = filter_var($data['description'], FILTER_SANITIZE_STRING);

    $component_id = (int) $data['component'];
    $component = $this->atlas->fetchRecord(Component::class, $component_id)
    $ticket->component = $component->getName();

    $this->atlas->persist($ticket);

    $response = $response->withRedirect("/tickets");
    return $response;
});
```

以上になります。

Atlasを最大限に活用する方法の詳細については、[公式ドキュメント](http://atlasphp.io/cassini/orm/)を必ずお読みください。

For more information on how to use Atlas to its greatest extent, be sure to
[read the official documentation](http://atlasphp.io/cassini/orm/):

- [マッパー間のリレーション定義](http://atlasphp.io/cassini/orm/relationships.html)

- [レコードとレコードセットの取得](http://atlasphp.io/cassini/orm/reading.html)

- [レコード](http://atlasphp.io/cassini/orm/records.html)と[レコードセット](http://atlasphp.io/cassini/orm/record-sets.html)の使用方法

- [トランザクション管理](http://atlasphp.io/cassini/orm/transactions.html)

- [ビヘイビアの追加](http://atlasphp.io/cassini/orm/behavior.html)

- [イベントハンドラー](http://atlasphp.io/cassini/orm/events.html)

- [ローレベルのダイレクトクエリ](http://atlasphp.io/cassini/orm/direct.html)

- カスタムマッパーメソッド、シングルテーブルの継承、多対多のリレーション、自動検証など[その他のトピック](http://atlasphp.io/cassini/orm/other.html)
