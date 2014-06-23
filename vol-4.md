Laravel Meetup Tokyo Vol.4
==========================================
#目次
[事前準備](#header1)  
[Laravelインストール](#header2)  
#Hands On
初めて触る方向けや、  
モデル、カスタムフィルター、カスタムバリデート、コンテナ周りまでの  
簡単な内容を予定しています。

##事前準備 {#header1} 
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

##Laravelインストール {#header2} 
今回は、公式laravelフレームワークをインストールします  
設置するディレクトリで以下の様にコマンドを実行して下さい  
```bash
#composerのパスを通している場合はこちら
$ composer create-project laravel/laravel プロジェクト名(英語で記述) --prefer-dist
#composerのパスを通していない場合は指定して実行して下さい
$ /パス/composer.php create-project laravel/laravel プロジェクト名(英語で記述) --prefer-dist
```
