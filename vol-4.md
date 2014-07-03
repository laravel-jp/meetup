Laravel Meetup Tokyo Vol.4
==========================================
#目次
[事前準備](#事前準備)  
[Laravelインストール](#laravelインストール)  
[helloLaravel](#hellolaravel)  
[はじめてのルーター](#はじめてのルーター)  
[はじめてのコントローラー](#はじめてのコントローラー)  
[はじめてのバリデート](#はじめてのバリデート)  
[はじめてのカスタムバリデート](#はじめてのカスタムバリデート)  
[はじめてのモデル](#はじめてのモデル)  
[はじめてのフィルター](#はじめてのフィルター)  

#Hands On
初めて触る方向けに、  
**モデル(Stub)**  
**カスタムフィルター**  
**カスタムバリデート**  
**コンテナ周り**  

までの簡単な内容を予定しています。

#事前準備
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

#Laravelインストール
今回は、公式laravelフレームワークをインストールします  
設置するディレクトリで以下の様にコマンドを実行して下さい  
```bash
#composerのパスを通している場合はこちら
$ composer create-project laravel/laravel プロジェクト名(英語で記述) --prefer-dist
#composerのパスを通していない場合は指定して実行して下さい
$ php /パス/composer.php create-project laravel/laravel プロジェクト名(英語で記述) --prefer-dist
```
**以降は$ php composer.pharではなく$ composerと記します**
インストール完了までしばしお待ちください
###実行権限
app/storage 配下にsessionやテンプレートのcacheファイル等が出力されるため、  
実行権限を与えて下さい
```bash
$ chmod -R 777 app/storage
```
###デバッグを有効にします
Laravelはhostnameで取得した名前でそれぞれの環境の動作を変更する事ができます。  
bootstrap/start.phpの**The Application Environment** 'local'に現在お使いのpcのhostnameを追加して下さい。  
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

#helloLaravel
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

#はじめてのルーター
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
**$_FILESは\Input::file()！**  
第二引数でデフォルト値を設定できます  

今設定されているルーターの情報は
```bash
$ php artisan routes
```
で確認できます  

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
        'name' => \Input::get('name', "Laravel!")
        // 配列に追加
    ];
    return \View::make('index', $array);
});
```
コントローラー要らずで、小さいアプリケーション等はこれだけでも十分実装できます

#はじめてのコントローラー
デフォルトで設置されているHomeControllerを使用します。  
先ほどのviewを使用しますので、下記の様に変更します
```php
public function showWelcome()
{
    return \View::make('index')
        ->with('name', \Input::get('name', "Laravel!"));
}
```
app/router.phpを次の様に変更します。  
```php
\Route::get('/', "HomeController@showWelcome");
```
ブラウザでアクセスしてみましょう  
**例 http://localhost:8000, http://localhost:8000/name=world**  

**/** 区切りでアクセスしたい場合は、  
app/router.phpを次の様に
```php
\Route::get('/{name?}', "HomeController@showWelcome");
```
HomeController.phpはこの様に
```php
public function showWelcome($name = "Laravel!")
{
    return \View::make('index')
        ->with('name', $name);
}
```
**例 http://localhost:8000, http://localhost:8000/world**  
routerだけで記述した場合とさほど変わりません  
routerとcontorollerの結びつけ方はこの他にも多数あります。  
[Laravel/docs/controllers日本語](http://laravel4.kore1server.com/docs/42/controllers)
#はじめてのバリデート
バリデートを簡単に使用してみましょう！  
先ほどまでのものをそのままにして、  
バリデートのルールを設定してみます  

```php
public function showWelcome()
{

    $validator = \Validator::make(\Input::all(), ['name' => 'required']);
    // validate
    if ($validator->fails()) {
        return \Response::json($validator->messages(), 500);
    }
    return \View::make('index')
        ->with('name', \Input::get('name', "Laravel!"));
}
```
nameを必須としました  
nullの場合は、せっかくなので簡単にjsonで返却する様にしてみました  
http://localhost:8000 にアクセスしてみましょう  
jsonで表示されています  
エラーの文字列も変えてしまいましょう  
エラーの文字列の変更の仕方はいくつかありますが、  
ここではvalidatorで直接文字列を指定します  
```php
public function showWelcome()
{
    $validator = \Validator::make(\Input::all(),
        ['name' => 'required'],
        ['name.required' => 'nameが空！']
    );
    // validate
    if ($validator->fails()) {
        return \Response::json($validator->messages(), 500);
    }
    return \View::make('index')
        ->with('name', \Input::get('name', "Laravel!"));
}
```
http://localhost:8000 にアクセスしてみましょう
変更されました  
nameに何か文字列を入れてバリデートが通る事を確認しましょう  
>validateはどこに記述しても構いません  
>コントローラーではなくモデルでもどこでもお好きなところに記述して下さい!  

折角なので'Laravel'以外は通らない様にしてみましょう
#はじめてのカスタムバリデート
laravelというルールを作ります

```php
public function showWelcome()
{
    \Validator::extend('laravel', function($attribute, $value, $parameters) {
        return ($value != 'Laravel') ? false : true;
    });

    $validator = \Validator::make(\Input::all(),
        ['name' => ['required|laravel']],
        [
            'name.required' => 'nameが空！',
            'name.laravel' => 'Laravelじゃありません！',
        ]
    );
    // validate
    if ($validator->fails()) {
        return \Response::json($validator->messages(), 500);
    }
    return \View::make('index')
        ->with('name', \Input::get('name', "Laravel!"));
}
```
対象に対して、複数適用するものがあれば、パイプで区切って指定する事も、  
配列にする事もできおます
```php
        $validator = \Validator::make(\Input::all(),
            ['name' => ['required', 'laravel']],
            [
                'name.required' => 'nameが空！',
                'name.laravel' => 'Laravelじゃありません！',
            ]
        );
```
デフォルトで様々なルールが用意されています。  
[バリデーション](http://laravel4.kore1server.com/docs/42/validation)
#はじめてのモデル
今回データベースは使用しませんが、  
下記の様なスタブを使ってcontorollerで使用してみましょう  
app/models配下にMeetUp.phpを作成します  
```php
<?php
namespace Models;

class MeetUp
{

    public function get()
    {
        return [
            'Laravel',
            'Symfony',
            'CakePHP'
        ];
    }
}
```
配列だけを返却するシンプルなものです  

コントローラーからこのクラスを使用するには、  
composerのautoloaderに登録しなければなりません  
composer.jsonを設置しているディレクトリで実行します
```php
$ composer dump-autoload
```
**Laravelのcomposer.jsonは、デフォルトのオートローラーはclassmapになっています  
そのためクラスを追加した場合は必ずdump-autoloadする必要があります。**
```json
	"autoload": {
		"classmap": [
			"app/commands",
			"app/controllers",
			"app/models",
			"app/database/migrations",
			"app/database/seeds",
			"app/tests/TestCase.php"
		]
	},
```
これ以外のディレクトリをオートローダーに含める場合は必ず追記しなければなりません
```json
  "autoload": {
    "classmap": [
      "app/commands",
      "app/controllers",
      "app/models",
      "app/database/migrations",
      "app/database/seeds",
      "app/tests/TestCase.php",
      "追加したディレクトリ"
    ]
  },
```
PHPのコーディング規約 PSRに変更する事でdump-autoloadは不要になりますので、  
慣れてきたら変更してみて下さい  
Laravelのディレクトリ構造はユーザーの好みでほとんどが自由に変更する事ができます  
bootstrap配下やconfig配下のソースを見てみましょう  

composerに登録したら、コントローラーです  
先ほどのHomeController.phpを使用します  
折角ですが先ほどのバリデートはコメントアウトです  
コンストラクタをこのようにします
```php

    /**
     * @param \Models\MeetUp $meetup
     */
    public function __construct(\Models\MeetUp $meetup)
    {
        $this->meetup = $meetup;
    }
```
タイプヒントで使用したいクラスを記述するだけです！！  
実際にDBや何かしらのデータストレージを使う場合も、  
コントローラーに直接 データベースのDBファサードを記述すると、  
DBに接続している状態でなければ実行できない(依存)コントローラーになります。  
Fatなコントローラーの誕生です!!!  
**ものすごく小さなアプリケーションの場合はそこまで神経質にならなくてもかまいません**  

下記の様にして、ブラウザなどから**http://localhost:8000**にアクセスしてみましょう
```php
class HomeController extends BaseController
{
    /** @var Models\MeetUp  */
    public $meetup;

    /**
     * @param \Models\MeetUp $meetup
     */
    public function __construct(\Models\MeetUp $meetup)
    {
        $this->meetup = $meetup;
    }


    public function showWelcome()
    {
        return \Response::json($this->meetup->get());
    }

}
```
折角なのでもう少しLaravelらしくしてみましょう  
MeetUpにインタフェースを継承させてみます  
インターフェースは、継承するクラスに、  
**このメソッドを実装しましょう！**というクラスの設計図みたいなものです  

app/models/MeetUpInterface.phpを作成して下記の様にします  
```php
<?php
namespace Models;

interface MeetUpInterface
{

    /**
     * @return mixed
     */
    public function get();
}
```
さきほどのMeetUpクラスでインターフェースを継承させます  
```php
<?php
namespace Models;

class MeetUp implements MeetUpInterface
{

    public function get()
    {
        return [
            'Laravel',
            'Symfony',
            'CakePHP'
        ];
    }
}
```
続いてHomeController.phpのコンストラクタを下記に変更します

```php
    /**
     * @param \Models\MeetUpInterface $meetup
     */
    public function __construct(\Models\MeetUpInterface $meetup)
    {
        $this->meetup = $meetup;
    }
```
```bash
$ composer dump-autoload
```
を忘れずに実行します  
そのままアクセスするとinterfaceはインスタンスを生成する事ができないため、当然エラーになります  

##インターフェースと実装を結びつけよう！
app/start/global.phpに下記のものを追記してみましょう  
どこでもかまいません  
```php
\App::bind('Models\\MeetUpInterface', 'Models\\MeetUp');
```
**http://localhost:8000**にアクセスしてみましょう  
先ほどの値が表示されているはずです  
このように、直接インスタンスを生成するのではなく、  
どこか外で定義して実行時に解決することを(依存性の注入)DI(dependency injection)といいます！  
LaravelのIocコンテナがよろしくやってくれます  
現在のstubからデータベースに変更する場合でも、  
同じinterfaceを継承してbindで変更するだけです！  
コントローラーなど手を入れる所はどこにもありません！  
更にテストも簡単です！  
他にも機能は色々あります  
さきほどのMeetUpクラスはこのようにしてインスタンスを生成する事もできます  
```php
    public function __construct()
    {
        $this->meetup = \App::make("Models\\MeetUp");
    }
```
**慣れてきたらglobal.phpではなく、ServiceProviderについて学んでみましょう**  
[IoCコンテナ](http://laravel4.kore1server.com/docs/ioc)
[Laravel4、IoCコンテナの魔術](http://kore1server.com/146)
[Laravel IoC コンテナの使い方](http://www.1x1.jp/blog/2014/02/how-to-use-ioc-container-in-laravel.html)
#はじめてのフィルター
最後にフィルターを少し触ってみましょう  
Laravelはコントローラーでそれぞれのuriの処理の前後に処理を挟む事ができます  
認証していないユーザーのアクセスを制限したり、  
IPアドレスで制限したり、  
色々な処理を実装する事ができます  
app/filter.phpに最初から用意されているものがあります  

下記のフィルターを追記してみます
```php
Route::filter('laravel', function() {
    if (Input::get('name') != "Laravel") {
        return Response::make("oh my god!", 503);
    }
});
```
引数nameがLaravel以外だと503を返します  
/app/routes.phpを下記の様に変更してみましょう
```php
\Route::get('/', ['uses' => 'HomeController@showWelcome', 'before' => ['laravel']]);
```
さらにroutesでフィルターを指定すると
```bash
$ php artisan routes
```
でルーターと一緒に確認する事もできます  
http://localhost:8000  
http://localhost:8000/?name=symfony  
http://localhost:8000/?name=Laravel  
それぞれアクセスして確認してみましょう  

フィルターの指定はroutes.phpに記述する以外に、  
コントローラーで指定する事もできます  
```php
    public function __construct(\Models\MeetUpInterface $meetup)
    {
        $this->meetup = $meetup;
        $this->beforeFilter('laravel', ['only' => ['showWelcome']]);
    }
```
フィルター指定方法はいくつかありますので、  
好みのものを使用してみて下さい  
[ルーティング](http://laravel4.kore1server.com/docs/routing)
