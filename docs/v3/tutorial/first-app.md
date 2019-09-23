---
title: First Application Walkthrough
---

もしあなたが非常にシンプルなスリムアプリケーション（Twigを使用せず、MonologとPDOデータベース接続を使用する）をセットアップする方法を探しているなら、このドキュメントは最適です。チュートリアルを進めてサンプルアプリケーションを構築するか、詳しく知りたい箇所が決まっているなら対象のステップを参照してください。

始める前に：サンプルアプリで始められる[skeleton project](https://github.com/slimphp/Slim-Skeleton) が提供されています。そのため、すべての機能の詳細が知りたいのではなく、とりあえずSlimアプリを動かしたい場合はこちらがおすすめです。

> このチュートリアルでは一連のアプリケーションを構築する一連の作業を紹介します。Slimのソースコードを読みたい場合は[こちら](https://github.com/slimphp/Tutorial-First-Application)から参照できます。

## Getting Set Up

プロジェクトのフォルダを作成することから始めます（ここではプロジェクトと名付けます。名前を付けるのって難しいですよね）。私はトップレベルにはコードではないもを配置します。そしてソースコード用のフォルダーとその中にWebのルートとなるソースコードを配置します。最初のディレクトリ配置は以下です。

```
.
├── project
│   └── src
│       └── public
```

### Installing Slim

[Composer](https://getcomposer.org)は、Slim Frameworkをインストールする最良の方法です。まだインストールしていない場合は、[インストール手順](https://getcomposer.org/download/)を参考にしてください。私のプロジェクトでは、`composer.phar`を`src/`ディレクトリにダウンロードしローカルで使用します。したがって、最初のコマンドは次のようになります（`src/`ディレクトリに移動してください）。

    php composer.phar require slim/slim

これは2つのことを行います。

- Slimフレームワークの依存関係を`composer.json`に追加します（私の場合、まだ持っていないのでcomposer.jsonファイルが作成されます。既にある状態で実行しても問題ありません）。

- `composer install`を実行して、それらの依存関係があるライブラリがアプリケーションで実際に使用できるようにします。

ここでプロジェクトディレクトリ内を見ると、すべてのライブラリコードを含む`vendor/`フォルダーがあることがわかります。 また`composer.json`と`composer.lock`の2つのファイルも作成されています。
これはソースコントロールのセットアップが正しく行えていることになります。composerを使用する際は常に`vendor/`ディレクトリを除きますが、`composer.json`と`composer.lock`はソースコントロールに含める必要があります。
このディレクトリでは`composer.phar`を使用しているのでリポジトリにも追加します。これにより`composer`コマンドを扱える他のシステムに同じようにインストールすることができます。

git ignoreを正しく設定するには、`src/.gitignore`というファイルを作成し、次の1行をファイルに追加します。

    vendor/*

gitは`vendor/`のファイルをリポジトリに追加することを推奨していません。依存関係はリポジトリによる管理ではなくcomposerに管理させるべきだからです。

### Create The Application

Slimの[サイト](http://www.slimframework.com)には、Slimフレームワークのシンプルかつ優れた`index.php`の例があるため、これをスタートとして使用します。次のコードを`src/public/index.php`に追加してください。

```php
<?php
use \Psr\Http\Message\ServerRequestInterface as Request;
use \Psr\Http\Message\ResponseInterface as Response;

require '../../vendor/autoload.php';

$app = new \Slim\App;
$app->get('/hello/{name}', function (Request $request, Response $response, array $args) {
    $name = $args['name'];
    $response->getBody()->write("Hello, $name");

    return $response;
});
$app->run();
```

この大量のコードをコピペするだけです... どのように処理されるか見てみましょう。

スクリプトの先頭にある`use`文で、スクリプトに`Request`クラスと`Response`クラスを取り込んでいるので、以降はクラスパスでそれらを記述する必要はありません。 Slimフレームワークは、HTTPメッセージングのPHP標準であるPSR-7をサポートしているので、アプリケーションをビルドすると、`Request`オブジェクトと`Response`オブジェクトが頻繁に表示されることに気付くでしょう。 これは、Webアプリケーションを作成するための最新の優れたアプローチです。

次に`vendor/autoload.php`ファイルをインクルードします。これはComposerによって作成され、Slimおよび事前にインストールした関連する依存関係を参照できるようになります。私と同じファイル構造を使用している場合、`vendor/`ディレクトリは`index.php`より上の階層にあり、上記のようにパスを修正する必要があるので注意してください。

最後に、Slimの恩恵を受けるスタートとなる`$app`オブジェクトを作成します。`$app->get()`は最初のルートです。`/hello/someone`にGETリクエストを送ると、これが反応することになります。最後の`$app->run()`の行が必要であることを忘れないでください。Slimに設定が完了し、メインイベントに取り掛かるタイミングであることを伝えます。

これでアプリケーションができまたので実行してみましょう。PHPのビルドインWebサーバーとApacheバーチャルホストのセットアップの2つのオプションについて説明します。

### PHPのWebサーバーでアプリケーションを実行

これは私が好む”クイックスタート”の方法です。なぜなら他の何にも依存しないからです。`src/public`ディレクトリからこのコマンドを実行してください。

    php -S localhost:8080

これにより、アプリケーションがhttp://localhost:8080で利用可能になります（マシンで既に8080ポートを使用している場合は警告が表示されます。
どのポートでも良いので空いている別のポート番号を選択してください）。

**注意** このURLだと「ページが見つかりません」というエラーメッセージが表示されるでしょう。ただしこれはSlimからのエラーメッセージであるため心配する必要はありません。
代わりにhttp://localhost:8080/hello/joebloggsを試してください)

### Apacheまたはnginxでアプリケーションを実行

これらを標準のLAMPスタックでセットアップするには、いくつか追加の設定が必要になります。バーチャルホストの設定と書き換えルールです。

バーチャルホストの設定はかなり簡単です。ここでは特別なものは必要ありません。既存のデフォルトのvhost設定をコピーし
`ServerName`を参照させたいあなたのプロジェクト名にセットしてください。たとえば次をように設定します。

    ServerName slimproject.test

    or for nginx:

    server_name slimproject.test;

次にプロジェクトの`/public`ディレクトリを指すようにDocumentRootを設定します（既存の行を編集します）。

    DocumentRoot    /home/lorna/projects/slim/project/src/public/

    or for nginx:

    root    /home/lorna/projects/slim/project/src/public/

設定を変更したら、サーバを**再起動**することを忘れないでください。

また私は`src/public`ディレクトリに`.htaccess`ファイルを作成しました。これはApacheの書き換えモジュールを有効にしているためで、
わたしたちの代わりにSlimがルーティングを行いすべてのWebリクエストを`index.php`に転送してくれます。これが私の`.htaccess`ファイルです。

```
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule . index.php [L]
```

nginxは`.htaccess`ファイルを使用しないため、`location`ブロックのサーバー設定に以下を追加してください。

```
if (!-e $request_filename){
    rewrite ^(.*)$ /index.php break;
}
```

*注意*: エントリポイントをindex.php以外のものにしたい場合は、設定も変更する必要があります。
`api.php`は一般にエントリポイントとしても使用されるため、それに応じて設定を一致させる必要があります。
ここでの例では、index.phpを使用していることを前提としています。

このチュートリアルでの設定は、他の例でよく使われるhttp://localhost:8080を、代わりにhttp://slimproject.testを使用
していることを忘れないでください。ただどちらも同じ警告が適用されます。http://slimproject.testにアクセスするとエラーページが表示されますが、
大切なのはSlimが発生させたエラーページだということです。 http://slimproject.test/hello/joebloggsにアクセスすると良い結果が見れるはずです。

## Configuration and Autoloaders

これでプラットフォームのセットアップが完了し、必要なものすべてをアプリケーションに配置できるようになりました。

### Add Config Settings to Your Application

最初の例では、すべてSlimのデフォルトを使用していますが、アプリケーションの作成時に設定を簡単に追加できます。
いくつかの選択肢がありますが、ここでは設定オプションの配列を作成し、Slimがここから設定オプションを取得するにしました。

これが初期設定です

```php
$config['displayErrorDetails'] = true;
$config['addContentLengthHeader'] = false;

$config['db']['host']   = 'localhost';
$config['db']['user']   = 'user';
$config['db']['pass']   = 'password';
$config['db']['dbname'] = 'exampleapp';
```

これらのコードは、`$app = new \Slim\App`の行の前に`src/public/index.php`に追加する必要があります。
最初の行が最も重要です！開発環境ではこれをオンにして、エラーに関する情報を取得します（これをしない場合、Slimは少なくともエラーを
ログに記録するため、PHPのビルドインWebサーバーを使用している場合は有益な情報がコンソールに出力されます）。
2行目では、WebサーバがContent-Lengthヘッダーを設定して、Slimの振る舞いを予測しやすくします。

他の設定は特定のキー/バリューの仕様ではなく、後で利用できるようにしたいデータです。

これをSlimに反映するには、以下のようにSlim/Appオブジェクトを作成する場所を次のように変更する必要があります。

```php
$app = new \Slim\App(['settings' => $config]);
```

`$config`配列に入れた設定は、後でアプリケーションからアクセスできるようになります。

## Set up Autoloading for Your Own Classes

Composerはvendorのクラスだけでなく、独自に作成したクラスのオートロードも処理できます。詳細なガイドについては、[Composerのオートローディングルールの管理](https://getcomposer.org/doc/04-schema.md#autoload)をご覧ください。

私のセットアップはとてもシンプルです。いくつか追加したクラスのファイルが`src/classes/`ディレクトリにあります。それらをオートロードするには、この`autoload`セクションを`composer.json`ファイルに追加します。

```javascript
{
    "require": {
        "slim/slim": "^3.1",
        "slim/php-view": "^2.0",
        "monolog/monolog": "^1.17",
        "robmorgan/phinx": "^0.5.1"
    },
    "autoload": {
        "psr-4": {
            "Example\\": "classes/"
        }
    }
}
```

## Add Dependencies

ほとんどのアプリケーションにはいくつかの依存関係を持ち、Slimは[Pimple](http://pimple.sensiolabs.org/)に構築されたDIC（Dependency Injection Container）を利用して適切に処理します。今回の例では、[Monolog](https://github.com/Seldaek/monolog)と[PDO](http://php.net/manual/en/book.pdo.php)でのMySQL接続を紹介します。

DIコンテナの考え方は、アプリケーションが必要した依存関係を、必要なときに読み込めるようにコンテナを構成することです。
DICは、1度作成/構築されると、それらを保存し必要に応じて後で再度使用できます。

コンテナを取得するには、以下のコードを`$app`を作成する行の後、かつアプリケーションをルートに登録する前のタイミングで追加できます。

```php
$container = $app->getContainer();
```

これで`Slim\Container`オブジェクトができたのでサービスに追加できます。

### Use Monolog In Your Application

Monologをよく知らない人もいると思いますが、なぜSlimがMonologを採用したかというと、それがPHPアプリケーションの優れたロギングフレームワークであるためです。まず、まず最初にComposerでMonologをインストールしましょう。

    php composer.phar require monolog/monolog

次のようにdependencyへは`logger`という名前で追加します。

```php
$container['logger'] = function($c) {
    $logger = new \Monolog\Logger('my_logger');
    $file_handler = new \Monolog\Handler\StreamHandler('../logs/app.log');
    $logger->pushHandler($file_handler);
    return $logger;
};
```

無名関数である要素をコンテナに追加します。（渡される`$c`はコンテナ本体なので、必要に応じて他のdependencyにアクセスできます）。そしてこの要素は初めてアクセスしようとしたときに呼び出されます。このコードは依存関係のセットアップを行います。次に同じdependencyにアクセスしようとすると、最初に作成された同じオブジェクトが使用されます。

ここでの私のMonologの設定はかなり簡易的です。すべてのエラーを`logs/app.log`というファイルに記録するように設定しているだけです（このパスは、スクリプトが実行されている場所、つまり`index.php`の観点からであることを注意してください）。

loggerを設定したら、ルート内から次のようなコードで使用できます。

```php
    $this->logger->addInfo('Something interesting happened');
```

優れたロギングは、あらゆるアプリケーションにとって非常に重要になるため、このように常にlogを記録することをお勧めします。これにより、必要なだけデバッグを追加できます。また各メッセージで適切なログレベルを使用することで、あらゆる時点で何をしているのかについて詳細を取得できます。

### Add A Database Connection

PHPで使用できるDBライブラリは多数ありますが、この例ではPDOを使用します。これはPHPで標準として使用できるため、おそらくすべてのプロジェクトで役立ちます。もしくは独自のライブラリを以下の例のように適用して使用できます

MonologをDICに追加した場合と同じように、無名関数（この場合は`db`と名付けます）をdependencyに追加します。

```php
$container['db'] = function ($c) {
    $db = $c['settings']['db'];
    $pdo = new PDO('mysql:host=' . $db['host'] . ';dbname=' . $db['dbname'],
        $db['user'], $db['pass']);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    $pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
    return $pdo;
};
```

以前にアプリに追加した設定を覚えていますか？これに私たちは設定を行いました。コンテナは設定したデータにアクセスする方法を知っているので、
ここから設定を簡単に取得できます。設定によりPDOオブジェクトを作成してデータベースに接続できるようになります。
（覚えておきたいのは、DB接続に失敗してPDOExceptionがスローされた場合はここで修正を行う必要があることです）。
必須ではないですが、2つの`setAttribute()`を設定に加えましたが。これら2つの設定によりPDO自体がライブラリとしてとても使いやすくなる
ので、この設定を例のように追加することをお勧めします。これでコネクションオブジェクトを取得できます。

繰り返しますが、`$this->`だけでdependenciesにアクセスでき必要な依存関係の名前（この場合は`$this->db`）を使用できます。
したがって次のようなコードになります。

```php
    $mapper = new TicketMapper($this->db);
```

これによりDICから`db`dependencyを取得し、必要に応じて作成されます。この例では`PDO`オブジェクトをマッパークラスに直接渡すことができます。

## Create Routes

`route`は機能をURLに添付するURLパターンです。 SlimはオートマッピングやURL変換式を使用しないため、好きなルートパターンを
好きな機能にマッピングでき非常に柔軟です。ルートは特定のHTTPメソッド（GETやPOSTなど）または複数のメソッドにリンクできます。

最初の例として、バグトラッカーサンプルアプリケーションのチケット一覧を取得する`/tickets`にGETメソッドをリクエストするコードを示します。
アプリケーションはまだ画面を作成していないため、ここではただ変数を返却します。

```php
$app->get('/tickets', function (Request $request, Response $response) {
    $this->logger->addInfo("Ticket list");
    $mapper = new TicketMapper($this->db);
    $tickets = $mapper->getTickets();

    $response->getBody()->write(var_export($tickets, true));
    return $response;
});
```

ここで`$app->get()`を使用すると、このルートはGETリクエストのみ利用可能になります。同等の`$app->post()`呼び出しもあり、
これもルートパターンとPOSTリクエストのコールバックを受け取ります。
また[他のHTTPメソッド](http://www.slimframework.com/docs/v3/objects/router.html)もありますし、
複数のメソッドが同じルートで同じコードを使用する必要がある場合の`map()`メソッドもあります。

Slimのルートは上からパターンに一致するため、別のルートと重複する可能性がある場合は、詳細なルートパターンを最初に配置する必要があり、
なにか問題があった場合Slimは例外をスローします。たとえば`/ticket/new`と`/ticket/{id}`の2つのパターンが設定されている場合、この順序で宣言する必要があります。
もし逆に`/ticket/{id}`が先だと、Slimのルーティングは"new"がIDであると見なしてしまいます!

このサンプルでは、すべてのルートをindex.phpに設定していますが、実際でこれをしてしまうと、かなり大きく扱いにくいファイルになってしまいます！
ルートを別のファイルに配置するようにリファクタリングするか、実際には別の場所で定義されている場所へのコールバックを設定したルートだけを登録するなどしてください。

すべてのルートコールバックは3つのパラメーターを受け入れます（3番目のパラメータはオプションです）。

 - Request: これには受信したリクエスト、ヘッダー、変数などに関するすべての情報が含まれます。
 - Response: レスポンスにはアウトプットとヘッダーを追加できます。完了するとクライアントが受信するHTTPレスポンスに変換されます。
 - Arguments: URLからの名前付きプレースホルダー（詳細は後ほど）、これはオプションであり、必要ない場合は通常省略します。

リクエストとレスポンスの重要な点は、Slim3がPSR-7標準のHTTPメッセージングに基づいていることです。
リクエストとレスポンスオブジェクトを使用すると、実際のリクエストとレスポンスを作成する必要がないため、
テストが容易になります。必要に応じてオブジェクトを設定するだけです。

### Routes with Named Placeholders

URLには、アプリケーションで使用するための変数が含まれている場合があります。
私のバグトラッカーを例とすると、チケットを参照するために'/ticket/42'などのURLが必要になります。Slimにはこの「42」を抽出してコードで
使用できるようにする簡単な方法があります。以下がそれを行うルーティングになります。
```php
$app->get('/ticket/{id}', function (Request $request, Response $response, $args) {
    $ticket_id = (int)$args['id'];
    $mapper = new TicketMapper($this->db);
    $ticket = $mapper->getTicketById($ticket_id);

    $response->getBody()->write(var_export($ticket, true));
    return $response;
});
```

`/ticket/{id}`と書かれたルート定義を見てください。こうするとルートは`{id}`と宣言されているURLの部分を取得し、
コールバック内で`$args['id']`として使用可能になります。

### Using GET Parameters

GETとPOSTでは全く異なる方法でデータが送信されるため、SlimではRequestオブジェクトからそのデータを取得する方法も大きく異なります。

連想配列を返す`$request->getQueryParams()`を実行することで、リクエストからすべてのクエリパラメータを取得できます。
したがって、URL`/tickets？sort=date＆order=desc`の場合、次のような連想配列を取得します。

    ['sort' => 'date', 'order' => 'desc']

これらは、コールバック内で（パラメータの検証はもちろん行ってください）使用できます。

### Working with POST Data

受信したデータを処理したい場合、これはbodyで見つけることができます。 すでにURLからデータを解析する方法、および`$request->getQueryParams()`を
使用してGETリクエストから変数を取得する方法については説明しましたが、POSTデータについてはどうでしょうか。
POSTリクエストのデータはリクエストのbodyにあります。Slimには便利な形式で情報を取得しやすくするための優れたヘルパーが
組み込まれています。

Webフォームからのデータの場合、Slimはそれを配列に変換します。チケットのサンプルアプリケーションには、`title`と`description`の2つの
フィールドを送信するだけの新しいチケットを作成するためのフォームがあります。
そのデータを受け取るルートの最初の部分は次のとおりです。POSTルートでは`$app->get()`ではなく`$app->post()`を使用することに注意してください。

```php
$app->post('/ticket/new', function (Request $request, Response $response) {
    $data = $request->getParsedBody();
    $ticket_data = [];
    $ticket_data['title'] = filter_var($data['title'], FILTER_SANITIZE_STRING);
    $ticket_data['description'] = filter_var($data['description'], FILTER_SANITIZE_STRING);
    // ...
```

`$request->getParsedBody()`は、bodyのデータを処理する際に、Slimにリクエストとそのヘッダーの`Content-Type`を調べるように
要求します。この例では単なるフォームのPOSTであるため、結果の`$data`配列は`$ _POST`を使用して取得されるものと非常によく似ています。
そして、[filter](https://php.net/manual/en/book.filter.php) の拡張機能を使用してこのデータを使用してよいものかを事前に確認できます。

Slimの組み込みメソッドを使用する大きな利点は、異なるリクエストオブジェクトを注入してテストできることです。
`$_ POST`を直接使用する場合、それを行うことができません。

本当に便利なのは、たとえばAPIを構築している場合やAJAXエンドポイントを作成した場合など、WebフォームからではないPOSTリクエストを受信した
ときもそのデータ形式を扱うのは非常に簡単だということです。 `Content-Type`ヘッダーが正しく設定されている限り、SlimはJSONペイロードを
解析して配列にし、`$request->getParsedBody()`を使用してまったく同じ方法でデータにアクセスできます。

## Views and Templates

Slimには、使用するビューについて指定はありませんが、プラグインを使用するための準備ができているオプションがいくつかあります。
最良の選択は、Twigまたは標準のPHPのいずれかですが両者には長所と短所があります。
Twigに既に精通している場合、レイアウトなどの優れた機能を多数提供します。一方、Twigをよく知らない場合、
マイクロフレームワークプロジェクトに追加さらる学習曲線のオーバーヘッドが大きくなる可能性があります
単純な方法を探しているなら、PHPの標準ビューがあなたにぴったりかもしれません！このサンプルプロジェクトではPHPを選択しましたが、
Twigに慣れている場合はそちらを使用してください。基本はほとんど同じです。

PHPビューを使用するため、Composerを使用してこのDIをプロジェクトに追加する必要があります。コマンドは次のようになります
（すでに前にやりましたね）

    php composer.phar require slim/php-view

ビューをレンダリングを利用するには、まずビューを作成してDICに追加しアプリケーションで使用できるようにする必要があります。
必要なコードは`src/public/index.php`の上部近くにある他のDICが追加されている場所で、次のようになります。

```php
$container['view'] = new \Slim\Views\PhpRenderer('../templates/');
```

Now we have a `view` element in the DIC, and by default it will look for its templates in the `src/templates/` directory.  We can use it to render templates in our actions - here's the ticket list route again, this time including the call to pass data into the template and render it:

```php
$app->get('/tickets', function (Request $request, Response $response) {
    $this->logger->addInfo('Ticket list');
    $mapper = new TicketMapper($this->db);
    $tickets = $mapper->getTickets();

    $response = $this->view->render($response, 'tickets.phtml', ['tickets' => $tickets]);
    return $response;
});
```

The only new part here is the penultimate line where we set the `$response` variable.  Now that the `view` is in the DIC, we can refer to it as `$this->view`.  Calling `render()` needs us to supply three arguments: the `$response` to use, the template file (inside the default templates directory), and any data we want to pass in.  Response objects are *immutable* which means that the call to `render()` won't update the response object; instead it will return us a new object which is why it needs to be captured like this.  This is always true when you operate on the response object.

When passing the data to templates, you can add as many elements to the array as you want to make available in the template.  The keys of the array are the variables that the data will exist in once we get to the template itself.

As an example, here's a snippet from the template that displays the ticket list (i.e. the code from `src/templates/tickets.phtml` - which uses [Pure.css](http://purecss.io/) to help cover my lack of frontend skills):

```php
<h1>All Tickets</h1>

<p><a href="/ticket/new">Add new ticket</a></p>

<table class="pure-table">
    <tr>
        <th>Title</th>
        <th>Component</th>
        <th>Description</th>
        <th>Actions</th>
    </tr>

<?php foreach ($tickets as $ticket): ?>

    <tr>
        <td><?=$ticket->getTitle() ?></td>
        <td><?=$ticket->getComponent() ?></td>
        <td><?=$ticket->getShortDescription() ?> ...</td>
        <td>
            <a href="<?=$router->pathFor('ticket-detail', ['id' => $ticket->getId()])?>">view</a>
        </td>
    </tr>

<?php endforeach; ?>
</table>
```

In this case, `$tickets` is actually a `TicketEntity` class with getters and setters, but if you passed in an array, you'd be able to access it using array rather than object notation here.

Did you notice something fun going on with `$router->pathFor()` right at the end of the example?  Let's talk about named routes next :)

### Easy URL Building with Named Routes

When we create a route, we can give it a name by calling `->setName()` on the route object.  In this case, I am adding the name to the route that lets me view an individual ticket so that I can quickly create the right URL for a ticket by just giving the name of the route, so my code now looks something like this (just the changed bits shown here):

```php
$app->get('/ticket/{id}', function (Request $request, Response $response, $args) {
    // ...
})->setName('ticket-detail');
```

To use this in my template, I need to make the router available in the template that's going to want to create this URL, so I've amended the `tickets/` route to pass a router through to the template by changing the render line to look like this:

```php
    $response = $this->view->render($response, 'tickets.phtml', ['tickets' => $tickets, 'router' => $this->router]);
```

With the `/tickets/{id}` route having a friendly name, and the router now available in our template, this is what makes the `pathFor()` call in our template work.  By supplying the `id`, this gets used as a named placeholder in the URL pattern, and the correct URL for linking to that route with those values is created.  This feature is brilliant for readable template URLs and is even better if you ever need to change a URL format for any reason - no need to grep templates to see where it's used.  This approach is definitely recommended, especially for links you'll use a lot.

## Where Next?

This article gave a walkthrough of how to get set up with a simple application of your own, which I hope will let you get quickly started, see some working examples, and build something awesome.

From here, I'd recommend you take a look at the other parts of the project documentation for anything you need that wasn't already covered or that you want to see an alternative example of.  A great next step would be to take a look at the [Middleware](https://www.slimframework.com/docs/v3/concepts/middleware.html) section - this technique is how we layer up our application and add functionality such as authentication which can be applied to multiple routes.
