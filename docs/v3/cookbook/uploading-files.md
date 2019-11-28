---
title: Uploading files using POST forms
---

フォームのPOSTリクエストを使用してアップロードされたファイルは、`Request`オブジェクトの[`getUploadedFiles`](/docs/v3/objects/request.html#uploaded-files)メソッドで取得できます。

POSTリクエストを使用してファイルをアップロードする場合、ファイルアップロードフォームに属性`enctype="multipart/form-data"`があることを確認してください。そうでないと`getUploadedFiles()`は空の配列を返します。

同じ入力名に対して複数のファイルがアップロードされている場合は、HTMLの入力名の後に括弧を追加します。
そうでないと`getUploadedFiles()`によって入力名に対してアップロードされたファイルが1つだけ返されます。

以下は、単一および複数のファイルアップロードの両方を含むHTMLフォームの例です。

<figure markdown="1">
```php
<!-- enctype属性がmultipart/form-dataに設定されていることを確認してください -->
<form method="post" enctype="multipart/form-data">
    <!-- 単一ファイルのアップロード -->
    <p>
        <label>Add file (single): </label><br/>
        <input type="file" name="example1"/>
    </p>

    <!-- 同じインプット名に複数フィールドを設定する際は大括弧を使用してください -->
    <p>
        <label>Add files (up to 2): </label><br/>
        <input type="file" name="example2[]"/><br/>
        <input type="file" name="example2[]"/>
    </p>

    <!-- 単一のインプットフィールドでも、大括弧をしようして複数のファイルをアップロードできます -->
    <p>
        <label>Add files (multiple): </label><br/>
        <input type="file" name="example3[]" multiple="multiple"/>
    </p>

    <p>
        <input type="submit"/>
    </p>
</form>
```
<figcaption>Figure 1: Example HTML form for file uploads</figcaption>
</figure>

アップロードされたファイルは、`moveTo`メソッドを使用してディレクトリに移動できます。以下は、上記のHTMLフォームでアップロードされたファイルを処理するアプリケーションの例です。

<figure markdown="1">
```php
<?php

require_once __DIR__ . '/vendor/autoload.php';

use Slim\Http\Request;
use Slim\Http\Response;
use Slim\Http\UploadedFile;

$app = new \Slim\App();

$container = $app->getContainer();
$container['upload_directory'] = __DIR__ . '/uploads';

$app->post('/', function(Request $request, Response $response) {
    $directory = $this->get('upload_directory');

    $uploadedFiles = $request->getUploadedFiles();

    // 単一ファイルのアップロードの処理
    $uploadedFile = $uploadedFiles['example1'];
    if ($uploadedFile->getError() === UPLOAD_ERR_OK) {
        $filename = moveUploadedFile($directory, $uploadedFile);
        $response->write('uploaded ' . $filename . '<br/>');
    }


    // 同一キーで複数のインプットがある場合の処理
    foreach ($uploadedFiles['example2'] as $uploadedFile) {
        if ($uploadedFile->getError() === UPLOAD_ERR_OK) {
            $filename = moveUploadedFile($directory, $uploadedFile);
            $response->write('uploaded ' . $filename . '<br/>');
        }
    }

    // 単一のインプットで複数のファイルアップロードがある場合の処理
    foreach ($uploadedFiles['example3'] as $uploadedFile) {
        if ($uploadedFile->getError() === UPLOAD_ERR_OK) {
            $filename = moveUploadedFile($directory, $uploadedFile);
            $response->write('uploaded ' . $filename . '<br/>');
        }
    }

});
アップロードされたファイルをアップロードディレクトリに移動し、既存のアップロードされたファイルの上書きを避けるため一意の名前を割り当てます

/**
 * アップロードされたファイルをアップロードディレクトリに移動し、すでにアップロードされたファイルの
 * 上書きを避けるため一意の名前を割り当てます。
 *
 * @param string $directory ファイルの移動先のディレクトリ
 * @param UploadedFile $uploadedFile アップロードされたファイル
 * @return string 移動したファイルの名前
 */
function moveUploadedFile($directory, UploadedFile $uploadedFile)
{
    $extension = pathinfo($uploadedFile->getClientFilename(), PATHINFO_EXTENSION);
    $basename = bin2hex(random_bytes(8)); // see http://php.net/manual/en/function.random-bytes.php
    $filename = sprintf('%s.%0.8s', $basename, $extension);

    $uploadedFile->moveTo($directory . DIRECTORY_SEPARATOR . $filename);

    return $filename;
}

$app->run();
```
<figcaption>Figure 2: Example Slim application to handle the uploaded files</figcaption>
</figure>

See also
--------
* [https://akrabat.com/psr-7-file-uploads-in-slim-3/](https://akrabat.com/psr-7-file-uploads-in-slim-3/)
