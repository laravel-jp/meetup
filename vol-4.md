Laravel Meetup Tokyo Vol.4
==========================================
#目次
[事前準備](#事前準備)  
[Laravelインストール](#Laravelインストール)  
[hello Laravel](#hello Laravel)
#Hands On
初めて触る方向けや、  
モデル、カスタムフィルター、カスタムバリデート、コンテナ周りまでの  
簡単な内容を予定しています。

##事前準備
**php5.4環境構築**  
**ext-mcryptインストール**  
phpのパスを通しておくと便利です。  
####mac
MAMPやXAMPPをお使いの方は、  
macのデフォルトのphpよりも先に読み込まれる様にする必要があります。  
```bash
$ vi .bash_profile
export PATH=phpへのパス:$PATH
```
**デフォルトのphpを利用される方は、必ずphp5.4以上にアップデートして下さい**  

####windows
コンピュータ->プロパティ->詳細設定->環境変数->pathを選択→編集  
```
PATH=C:\デフォルトのもの;C:\phpへのパス;  
```
上記の様に指定して下さい。  

####vagrant
公式のものを利用すると便利です。  
[homestead](http://laravel.com/docs/homestead)  


ビルトインサーバで動かすので、apache, nginxなどのwebサーバは不要です  
また今回はDatabaseは使用しません。  
###composer インストール
```bash
$ curl -sS https://getcomposer.org/installer | php
```
curlが使用できない場合は、
```bash
$ php -r "readfile('https://getcomposer.org/installer');" | php
```
インストールするディレクトリを指定する場合は次の様に指定して下さい。  
```bash
$ curl -sS https://getcomposer.org/installer | php -- --install-dir=インストールしたいディレクトリ
```

すでに導入されている方は不要です。  
composer.pharへのパスを通します。  
```bash
# windowsの方はmove
$ mv composer.phar /usr/local/bin/composer(など任意の場所へ設置して下さい)
```
**不要の方は、/パス/composer.phpで利用して下さい。**

##Laravelインストール
今回は、公式laravelフレームワークをインストールします  
設置するディレクトリで以下の様にコマンドを実行して下さい  
```bash
#composerのパスを通している場合はこちら
$ composer create-project laravel/laravel プロジェクト名(英語で記述) --prefer-dist
#composerのパスを通していない場合は指定して実行して下さい
$ /パス/composer.php create-project laravel/laravel プロジェクト名(英語で記述) --prefer-dist
```
インストール完了までしばしお待ちください
###実行権限
app/storage 配下にsessionやテンプレートのcacheファイル等が出力されるため、  
実行権限を与えて下さい
```bash
$ chmod -R 777 app/storage
```
###デバッグを有効にします
Laravelはhostnameで取得した名前でそれぞれの環境の動作を変更する事ができます。 
'local'に現在お使いのpcのhostnameを追加して下さい。  
```php
$env = $app->detectEnvironment([
    'local' => ['homestead', 'hostnameで追加した名称'],
]);
```
こうする事で、app/config配下にlocalディレクトリがあれば、  
そこに配置されているコンフィグファイルを利用します。  
また変更したい箇所だけを記述したファイルでも構いません。  
ここではlocal配下に設置されている **app.php** を次の様に記述します。  

```php
return [
    // debugを有効に
    'debug' => true,
    // タイムゾーンを日本に
    'timezone' => 'Asia/Tokyo',
];

```
今回変更するものはこれだけです！  
database,session,cache等も同様に変更する事ができます

##hello Laravel
ビルトインサーバで実行します
```bash
$ php artisan serve
```
デフォルトはlocalhost:8000で起動します。  
ポートを変更する場合は次の様にしましょう
```bash
$ php artisan serve --port 8888
```
またlocalhostではなく、任意のhostを指定する事もできます。  
```bash
$ php artisan serve --host 192.168.1.1 --port 8888
```

##初めてのrouter
router.phpを以下の様に変更してみて下さい  
```php
\Route::get('/', function() {
    return "Hello Laravel!";
});
```
serveで起動したアドレスにアクセスしてください  
表示されましたか？
getとはHTTPのGETを指します  
getをpostに変更してみて下さい  
```php
\Route::post('/', function() {
    return "Hello Laravel!";
});
```
Exceptionが投げられたはずです。  
同様にput, patch, delete 等が指定できます。  
もう少し触ってみましょう。  
今度はLaravelの文字をuriで変更します  
```php
\Route::get('/{name?}', function($name = 'Laravel!') {
    return "Hello {$name}";
});
```
セグメントで指定していない場合は、先ほどと同じですが、  
任意の文字を入れてアクセスしてみて下さい  
**例 http://localhost:8000/world**  

?をつける事でnullが許容され、functionで指定された初期値が代入されます  
クエリーで値を受け取りたい時は、次の様にします。  
```php
\Route::get('/', function() {
    $name = \Input::get('name', "Laravel!");
    return "Hello {$name}";
});
```
http://localhost:8000にアクセスするとそのままで何も変わりません  
http://localhost:8000/?name=world としてアクセスするとnameが反映されます  
**\Input::get()は、$_GETでも$_POSTでも特に区別する事なく値を取得します**  
**$_FILESでも同じです！**  
第二引数でデフォルト値を設定できます  

###routerとview
bladeテンプレートを使って同じ事をしてみましょう  
app/views/index.blade.phpを作成してください  
中身は次の様にします
```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>{{$name}}</title>
</head>
<body>
    Hello {{$name}}
</body>
</html>
```
router.phpは先ほどものを以下の様に変更します  
```php
\Route::get('/', function() {
    return \View::make('index')->with('name', \Input::get('name', "Laravel!"));
    // name以外のものも同時に渡したい場合は 
    // return \View::make('index')->with('name', \Input::get('name', "Laravel!"))->with('hoge', 'hoge');
});
```
次の様に書く事もできます  
```php
\Route::get('/', function() {
    $array = [
        'name', \Input::get('name', "Laravel!")
        // 配列に追加
    ];
    return \View::make('index', $array);
});
```
コントローラー要らずで、小さいアプリケーション等はこれだけでも十分実装できます  























