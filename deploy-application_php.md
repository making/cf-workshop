## 簡単なアプリケーションをデプロイ (PHP編)

[Lumen](https://lumen.laravel.com/)を使ってとても簡単なWebアプリケーションを作成 & デプロイしましょう。

PHP、[Composer](https://getcomposer.org/download/)がインストール済みであることを前提としています。

### プロジェクトの作成

Composer `create-project`で`hello-cf`プロジェクトを作成してください。

``` console
$ composer create-project --prefer-dist laravel/lumen hello-cf
$ cd hello-cf
```

`app/Http/routes.php`を編集して、次の内容を記述してください。

``` php
<?php

/*
|--------------------------------------------------------------------------
| Application Routes
|--------------------------------------------------------------------------
|
| Here is where you can register all of the routes for an application.
| It is a breeze. Simply tell Lumen the URIs it should respond to
| and give it the Closure to call when that URI is requested.
|
*/

$app->get('/', function () use ($app) {
    return 'Hello World!'; // ここを変更
});
```


まずはローカルでアプリケーションを実行してみましょう。

``` console
$ php -S localhost:8080 -t public
```

[http://localhost:8080](http://localhost:8080)にアクセスしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/02020313-0016-ba4e-e2bd-f4ddd5db40f0.png)

Hello World!が表示されれば成功です。

### アプリケーションをCloud FoundryにPush

作成したアプリケーションをCloud FoundryにPushしましょう。

まずはCloud Foundry上で使用するPHPのバージョンとWebに公開するディレクトリを`.bp-config/options.json`に記述します。今回はPHP 7.0の最新版を使用します。

``` console
$ mkdir .bp-config
```

`options.json`に次の内容を記述してください。

``` json
{
    "WEBDIR": "public",
    "PHP_VERSION": "{PHP_70_LATEST}"
}
```

> **ノート: 指定できるPHPのバージョン**
> 
> 指定できるPHPのバージョン及びExtensionは使用するBuildpackによります。
> 
> ``` console
> $ cf buildpacks
> Getting buildpacks...
> 
> buildpack                    position   enabled   locked   filename
> staticfile_buildpack         1          true      true     staticfile_buildpack-cached-v1.3.10.zip
> java_buildpack               2          true      true     java-buildpack-offline-v3.9.zip
> ruby_buildpack               3          true      true     ruby_buildpack-cached-v1.6.24.zip
> nodejs_buildpack             4          true      true     nodejs_buildpack-cached-v1.5.19.zip
> go_buildpack                 5          true      true     go_buildpack-cached-v1.7.12.zip
> python_buildpack             6          true      true     python_buildpack-cached-v1.5.9.zip
> php_buildpack                7          true      true     php_buildpack-cached-v4.3.19.zip <-----
> liberty_buildpack            8          true      true     liberty_buildpack.zip
> binary_buildpack             9          true      true     binary_buildpack-cached-v1.0.3.zip
> dotnet_core_buildpack_beta   10         true      true     dotnet-core_buildpack-cached-v1.0.0.zip
> ```
> 
> 上の例ではPHPのBuildpackバージョンは4.3.19です。BuildpackのリリースページにそのBuildpackバージョンで利用出来るPHPのバージョン及びExtensionが列挙されています。
> 
> https://github.com/cloudfoundry/php-buildpack/releases/tag/v4.3.19
>
> `{PHP_70_LATEST}`と書くと、[defaults/options.json](https://github.com/cloudfoundry/php-buildpack/blob/v4.3.19/defaults/options.json#L14)に定義されたバージョンが使用されます。`composer.json`にもPHPバージョンが指定されていればそちらが優先されます。
>
> また、Extensionsは次のように指定できます。`bz2`, `zlib`, `curl`, `mcrypt`はデフォルトで有効になっています。
> 
> ```json
> {
>     "WEBDIR": "public",
>     "PHP_VERSION": "{PHP_70_LATEST}",
>     "PHP_EXTENSIONS": [ "bz2", "zlib", "curl", "mcrypt", "pdo", "pdo_mysql", "mbstring"]
> }
> ```
> 
> その他、`options.json`の設定方法は[ドキュメント](http://docs.cloudfoundry.org/buildpacks/php/gsg-php-config.html)を参照してください。


`cf`コマンドを使ってCloud Foundryにアプリケーションをデプロイできます。プロジェクトルートディレクトリで実行してください。

``` console
$ cf push hello-<your name> -m 64m
```

`<your name>`は自分の名前などを置換して、一意にしてください。以下では`<your name>`を`tmaki`とします。適宜自分の名前に読み替えてください。
`-m`で使用するメモリ量を指定します。Pivotal Web Servicesではデフォルトで1GBが使用されますが、今回のPHPアプリではそんなに必要ないのでここでは64MBを指定しています。

``` console
$ cf push hello-tmaki -m 64m
Creating app hello-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK

Creating route hello-tmaki.cfapps.io...
OK

Binding hello-tmaki.cfapps.io to hello-tmaki...
OK

Uploading hello-tmaki...
Uploading app files from: /Users/makit/cfws/php/hello-cf
Uploading 3.9M, 4209 files
Done uploading               
OK

Starting app hello-tmaki in org APJ / space staging as tmaki@pivotal.io...
Downloading liberty_buildpack...
Downloading php_buildpack...
Downloaded dotnet_core_buildpack_beta
Downloading java_buildpack...
Downloading dotnet_core_buildpack_beta...
Downloading binary_buildpack...
Downloaded go_buildpack
Downloaded ruby_buildpack
Downloaded binary_buildpack
Downloaded python_buildpack
Downloaded nodejs_buildpack
Downloading python_buildpack...
Downloaded java_buildpack
Creating container
Downloaded php_buildpack
Downloading ruby_buildpack...
Downloaded staticfile_buildpack
Downloaded liberty_buildpack
Successfully created container
Downloaded app package (5.8M)
Staging...
-------> Buildpack version 4.3.19
Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_httpd_httpd-2.4.23-linux-x64.tgz] to [/tmp]
WARNING: A version of PHP has been specified in both `composer.json` and `./bp-config/options.json`.
WARNING: The version defined in `composer.json` will be used.
Installing PHP
PHP 7.0.9
Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_php7_php7-7.0.9-linux-x64-1472493075.tgz] to [/tmp]
The extension 'tokenizer' is not provided by this buildpack.
The extension 'dom' is not provided by this buildpack.
The extension 'json' is not provided by this buildpack.
The extension 'pcre' is not provided by this buildpack.
The extension 'reflection' is not provided by this buildpack.
The extension 'spl' is not provided by this buildpack.
Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_php7_php7-7.0.9-linux-x64-1472493075.tgz] to [/tmp]
Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_php_binaries_trusty_composer_1.2.0_composer.phar] to [/tmp]
Installing dependencies from lock file
Loading composer repositories with package information
  - Installing doctrine/inflector (v1.1.0)
    Downloading
  - Installing symfony/polyfill-mbstring (v1.2.0)
    Downloading
  - Installing symfony/console (v3.0.9)
    Downloading
  - Installing symfony/translation (v3.0.9)
    Downloading
  - Installing nesbot/carbon (1.21.0)
    Downloading
  - Installing paragonie/random_compat (v1.4.1)
    Downloading
  - Installing illuminate/contracts (v5.2.45)
    Downloading
  - Installing illuminate/support (v5.2.45)
    Downloading
  - Installing illuminate/console (v5.2.45)
    Downloading
  - Installing symfony/http-foundation (v3.0.9)
    Downloading
  - Installing symfony/finder (v3.0.9)
    Downloading
  - Installing illuminate/session (v5.2.45)
    Downloading
  - Installing symfony/polyfill-util (v1.2.0)
    Downloading
  - Installing symfony/polyfill-php56 (v1.2.0)
    Downloading
  - Installing symfony/event-dispatcher (v3.1.4)
    Downloading
  - Installing psr/log (1.0.0)
    Downloading
  - Installing symfony/debug (v3.0.9)
    Downloading
  - Installing symfony/http-kernel (v3.0.9)
    Downloading
  - Installing nikic/fast-route (v0.7.0)
    Downloading
  - Installing mtdowling/cron-expression (v1.1.0)
    Downloading
  - Installing monolog/monolog (1.21.0)
    Downloading
  - Installing illuminate/filesystem (v5.2.45)
    Downloading
  - Installing illuminate/container (v5.2.45)
    Downloading
  - Installing illuminate/events (v5.2.45)
    Downloading
  - Installing illuminate/view (v5.2.45)
    Downloading
  - Installing illuminate/validation (v5.2.45)
    Downloading
  - Installing illuminate/translation (v5.2.45)
    Downloading
  - Installing symfony/process (v3.0.9)
    Downloading
  - Installing illuminate/queue (v5.2.45)
    Downloading
  - Installing illuminate/pipeline (v5.2.45)
    Downloading
  - Installing illuminate/pagination (v5.2.45)
    Downloading
  - Installing illuminate/http (v5.2.45)
    Downloading
  - Installing illuminate/hashing (v5.2.45)
    Downloading
  - Installing illuminate/encryption (v5.2.45)
    Downloading
  - Installing illuminate/database (v5.2.45)
    Downloading
  - Installing illuminate/config (v5.2.45)
    Downloading
  - Installing illuminate/cache (v5.2.45)
    Downloading
  - Installing illuminate/bus (v5.2.45)
    Downloading
  - Installing illuminate/broadcasting (v5.2.45)
    Downloading
  - Installing illuminate/auth (v5.2.45)
    Downloading
  - Installing laravel/lumen-framework (v5.2.9)
    Downloading
  - Installing vlucas/phpdotenv (v2.4.0)
    Downloading
Generating autoload files
Finished: [2016-09-16 04:59:45.354772]
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (1.5M)
Uploaded droplet (62.5M)
Uploading complete
Destroying container
Successfully destroyed container

1 of 1 instances running

App started


OK

App hello-tmaki was started using this command `$HOME/.bp/bin/start`

Showing health and status for app hello-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK

requested state: started
instances: 1/1
usage: 64M x 1 instances
urls: hello-tmaki.cfapps.io
last uploaded: Fri Sep 16 04:59:25 UTC 2016
stack: cflinuxfs2
buildpack: php 4.3.19

     state     since                    cpu    memory         disk           details
#0   running   2016-09-16 02:00:08 PM   0.0%   28.1M of 64M   208.6M of 1G
````

これでデプロイに成功しました。
`cf apps`でデプロイされているアプリケーションの一覧を取得できます。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name          requested state   instances   memory   disk   urls   
hello-tmaki   started           1/1         64M      1G     hello-tmaki.cfapps.io <---
```

`urls`の列に出力されている値がアプリケーションのURLです。この場合は[http://hello-tmaki.cfapps.io](http://hello-tmaki.cfapps.io)です。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/79860de1-4846-c922-8583-787e77a185d2.png)

Cloud Foundry上にデプロイされたアプリケーションにもアクセスできました。

[PWSの管理画面](https://console.run.pivotal.io)を見てみましょう。
![image](https://qiita-image-store.s3.amazonaws.com/0/1852/4fd1565a-9bad-63a2-9382-6e7f59363b10.png)

「Space development」をクリックしてください。`development`というスペースにデプロイされているアプリケーションの一覧を確認できます。`hello-<your name>`が表示されています。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/a0684f30-3757-2cfe-0580-c19b3598a499.png)

`hello-<your name>`をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/58280398-6b1b-1d41-7581-62ba76f24d06.png)

アプリケーションの情報が確認できます。

直近のログは`cf logs <App> --recent`で確認できます。

``` console
$ cf logs hello-tmaki --recent
Connected, dumping recent logs for app hello-tmaki in org APJ / space staging as tmaki@pivotal.io...

2016-09-16T13:58:51.37+0900 [API/4]      OUT Created app with guid 18f0966b-f3ba-4c5d-92e7-9714a1c40ac2
2016-09-16T13:58:55.74+0900 [API/1]      OUT Updated app with guid 18f0966b-f3ba-4c5d-92e7-9714a1c40ac2 ({"route"=>"ae06e3e0-1ca3-4baf-97cd-254824ab4088", :verb=>"add", :relation=>:routes, :related_guid=>"ae06e3e0-1ca3-4baf-97cd-254824ab4088"})
2016-09-16T13:59:31.46+0900 [API/2]      OUT Updated app with guid 18f0966b-f3ba-4c5d-92e7-9714a1c40ac2 ({"state"=>"STARTED"})
2016-09-16T13:59:31.96+0900 [STG/0]      OUT Downloading dotnet_core_buildpack_beta...
2016-09-16T13:59:31.96+0900 [STG/0]      OUT Downloading liberty_buildpack...
2016-09-16T13:59:31.96+0900 [STG/0]      OUT Downloading python_buildpack...
2016-09-16T13:59:31.96+0900 [STG/0]      OUT Downloading php_buildpack...
2016-09-16T13:59:31.99+0900 [STG/0]      OUT Downloaded dotnet_core_buildpack_beta
2016-09-16T13:59:31.99+0900 [STG/0]      OUT Downloading binary_buildpack...
2016-09-16T13:59:31.99+0900 [STG/0]      OUT Downloaded php_buildpack
2016-09-16T13:59:31.99+0900 [STG/0]      OUT Downloading java_buildpack...
2016-09-16T13:59:32.01+0900 [STG/0]      OUT Downloaded go_buildpack
2016-09-16T13:59:32.01+0900 [STG/0]      OUT Downloading ruby_buildpack...
2016-09-16T13:59:32.02+0900 [STG/0]      OUT Downloaded ruby_buildpack
2016-09-16T13:59:32.03+0900 [STG/0]      OUT Downloaded binary_buildpack
2016-09-16T13:59:32.04+0900 [STG/0]      OUT Downloaded nodejs_buildpack
2016-09-16T13:59:32.06+0900 [STG/0]      OUT Downloaded staticfile_buildpack
2016-09-16T13:59:32.10+0900 [STG/0]      OUT Downloaded python_buildpack
2016-09-16T13:59:32.14+0900 [STG/0]      OUT Downloaded liberty_buildpack
2016-09-16T13:59:32.28+0900 [STG/0]      OUT Downloaded java_buildpack
2016-09-16T13:59:32.28+0900 [STG/0]      OUT Creating container
2016-09-16T13:59:33.11+0900 [STG/0]      OUT Successfully created container
2016-09-16T13:59:34.34+0900 [STG/0]      OUT Downloaded app package (5.8M)
2016-09-16T13:59:34.34+0900 [STG/0]      OUT Staging...
2016-09-16T13:59:35.93+0900 [STG/0]      OUT -------> Buildpack version 4.3.19
2016-09-16T13:59:36.12+0900 [STG/0]      OUT Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_httpd_httpd-2.4.23-linux-x64.tgz] to [/tmp]
2016-09-16T13:59:36.26+0900 [STG/0]      OUT WARNING: A version of PHP has been specified in both `composer.json` and `./bp-config/options.json`.
2016-09-16T13:59:36.26+0900 [STG/0]      OUT WARNING: The version defined in `composer.json` will be used.
2016-09-16T13:59:36.26+0900 [STG/0]      OUT Installing PHP
2016-09-16T13:59:36.26+0900 [STG/0]      OUT PHP 7.0.9
2016-09-16T13:59:36.76+0900 [STG/0]      OUT Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_php7_php7-7.0.9-linux-x64-1472493075.tgz] to [/tmp]
2016-09-16T13:59:38.15+0900 [STG/0]      ERR The extension 'tokenizer' is not provided by this buildpack.
2016-09-16T13:59:38.15+0900 [STG/0]      ERR The extension 'dom' is not provided by this buildpack.
2016-09-16T13:59:38.15+0900 [STG/0]      ERR The extension 'json' is not provided by this buildpack.
2016-09-16T13:59:38.15+0900 [STG/0]      ERR The extension 'pcre' is not provided by this buildpack.
2016-09-16T13:59:38.15+0900 [STG/0]      ERR The extension 'reflection' is not provided by this buildpack.
2016-09-16T13:59:38.15+0900 [STG/0]      ERR The extension 'spl' is not provided by this buildpack.
2016-09-16T13:59:38.58+0900 [STG/0]      OUT Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_php7_php7-7.0.9-linux-x64-1472493075.tgz] to [/tmp]
2016-09-16T13:59:40.27+0900 [STG/0]      OUT Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_php_binaries_trusty_composer_1.2.0_composer.phar] to [/tmp]
2016-09-16T13:59:40.53+0900 [STG/0]      ERR Loading composer repositories with package information
2016-09-16T13:59:40.53+0900 [STG/0]      ERR Installing dependencies from lock file
2016-09-16T13:59:40.54+0900 [STG/0]      ERR   - Installing doctrine/inflector (v1.1.0)
2016-09-16T13:59:40.54+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:40.63+0900 [STG/0]      ERR   - Installing symfony/polyfill-mbstring (v1.2.0)
2016-09-16T13:59:40.63+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:40.71+0900 [STG/0]      ERR   - Installing symfony/console (v3.0.9)
2016-09-16T13:59:40.71+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:40.84+0900 [STG/0]      ERR   - Installing symfony/translation (v3.0.9)
2016-09-16T13:59:40.84+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:40.95+0900 [STG/0]      ERR   - Installing nesbot/carbon (1.21.0)
2016-09-16T13:59:40.95+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:41.02+0900 [STG/0]      ERR   - Installing paragonie/random_compat (v1.4.1)
2016-09-16T13:59:41.03+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:41.10+0900 [STG/0]      ERR   - Installing illuminate/contracts (v5.2.45)
2016-09-16T13:59:41.10+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:41.21+0900 [STG/0]      ERR   - Installing illuminate/support (v5.2.45)
2016-09-16T13:59:41.21+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:41.31+0900 [STG/0]      ERR   - Installing illuminate/console (v5.2.45)
2016-09-16T13:59:41.31+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:41.41+0900 [STG/0]      ERR   - Installing symfony/http-foundation (v3.0.9)
2016-09-16T13:59:41.41+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:41.55+0900 [STG/0]      ERR   - Installing symfony/finder (v3.0.9)
2016-09-16T13:59:41.55+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:41.66+0900 [STG/0]      ERR   - Installing illuminate/session (v5.2.45)
2016-09-16T13:59:41.66+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:41.76+0900 [STG/0]      ERR   - Installing symfony/polyfill-util (v1.2.0)
2016-09-16T13:59:41.76+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:41.83+0900 [STG/0]      ERR   - Installing symfony/polyfill-php56 (v1.2.0)
2016-09-16T13:59:41.83+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:41.89+0900 [STG/0]      ERR   - Installing symfony/event-dispatcher (v3.1.4)
2016-09-16T13:59:41.89+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:41.97+0900 [STG/0]      ERR   - Installing psr/log (1.0.0)
2016-09-16T13:59:41.97+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:42.04+0900 [STG/0]      ERR   - Installing symfony/debug (v3.0.9)
2016-09-16T13:59:42.04+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:42.14+0900 [STG/0]      ERR   - Installing symfony/http-kernel (v3.0.9)
2016-09-16T13:59:42.14+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:42.29+0900 [STG/0]      ERR   - Installing nikic/fast-route (v0.7.0)
2016-09-16T13:59:42.29+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:42.39+0900 [STG/0]      ERR   - Installing mtdowling/cron-expression (v1.1.0)
2016-09-16T13:59:42.39+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:42.46+0900 [STG/0]      ERR   - Installing monolog/monolog (1.21.0)
2016-09-16T13:59:42.46+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:42.58+0900 [STG/0]      ERR   - Installing illuminate/filesystem (v5.2.45)
2016-09-16T13:59:42.58+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:42.67+0900 [STG/0]      ERR   - Installing illuminate/container (v5.2.45)
2016-09-16T13:59:42.67+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:42.74+0900 [STG/0]      ERR   - Installing illuminate/events (v5.2.45)
2016-09-16T13:59:42.74+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:42.81+0900 [STG/0]      ERR   - Installing illuminate/view (v5.2.45)
2016-09-16T13:59:42.81+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:42.88+0900 [STG/0]      ERR   - Installing illuminate/validation (v5.2.45)
2016-09-16T13:59:42.88+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:42.98+0900 [STG/0]      ERR   - Installing illuminate/translation (v5.2.45)
2016-09-16T13:59:42.98+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:43.07+0900 [STG/0]      ERR   - Installing symfony/process (v3.0.9)
2016-09-16T13:59:43.07+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:43.14+0900 [STG/0]      ERR   - Installing illuminate/queue (v5.2.45)
2016-09-16T13:59:43.14+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:43.24+0900 [STG/0]      ERR   - Installing illuminate/pipeline (v5.2.45)
2016-09-16T13:59:43.24+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:43.33+0900 [STG/0]      ERR   - Installing illuminate/pagination (v5.2.45)
2016-09-16T13:59:43.34+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:43.43+0900 [STG/0]      ERR   - Installing illuminate/http (v5.2.45)
2016-09-16T13:59:43.43+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:43.52+0900 [STG/0]      ERR   - Installing illuminate/hashing (v5.2.45)
2016-09-16T13:59:43.52+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:43.61+0900 [STG/0]      ERR   - Installing illuminate/encryption (v5.2.45)
2016-09-16T13:59:43.61+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:43.70+0900 [STG/0]      ERR   - Installing illuminate/database (v5.2.45)
2016-09-16T13:59:43.70+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:43.83+0900 [STG/0]      ERR   - Installing illuminate/config (v5.2.45)
2016-09-16T13:59:43.83+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:43.92+0900 [STG/0]      ERR   - Installing illuminate/cache (v5.2.45)
2016-09-16T13:59:43.92+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:44.01+0900 [STG/0]      ERR   - Installing illuminate/bus (v5.2.45)
2016-09-16T13:59:44.01+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:44.10+0900 [STG/0]      ERR   - Installing illuminate/broadcasting (v5.2.45)
2016-09-16T13:59:44.10+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:44.19+0900 [STG/0]      ERR   - Installing illuminate/auth (v5.2.45)
2016-09-16T13:59:44.19+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:44.29+0900 [STG/0]      ERR   - Installing laravel/lumen-framework (v5.2.9)
2016-09-16T13:59:44.29+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:44.39+0900 [STG/0]      ERR   - Installing vlucas/phpdotenv (v2.4.0)
2016-09-16T13:59:44.39+0900 [STG/0]      ERR     Downloading
2016-09-16T13:59:45.11+0900 [STG/0]      ERR Generating autoload files
2016-09-16T13:59:45.35+0900 [STG/0]      OUT Finished: [2016-09-16 04:59:45.354772]
2016-09-16T13:59:57.33+0900 [STG/0]      OUT Exit status 0
2016-09-16T13:59:57.33+0900 [STG/0]      OUT Staging complete
2016-09-16T13:59:57.33+0900 [STG/0]      OUT Uploading droplet, build artifacts cache...
2016-09-16T13:59:57.33+0900 [STG/0]      OUT Uploading droplet...
2016-09-16T13:59:57.33+0900 [STG/0]      OUT Uploading build artifacts cache...
2016-09-16T13:59:57.44+0900 [STG/0]      OUT Uploaded build artifacts cache (1.5M)
2016-09-16T14:00:01.47+0900 [STG/0]      OUT Uploaded droplet (62.5M)
2016-09-16T14:00:01.48+0900 [STG/0]      OUT Uploading complete
2016-09-16T14:00:01.54+0900 [STG/0]      OUT Destroying container
2016-09-16T14:00:02.31+0900 [CELL/0]     OUT Creating container
2016-09-16T14:00:03.29+0900 [STG/0]      OUT Successfully destroyed container
2016-09-16T14:00:03.79+0900 [CELL/0]     OUT Successfully created container
2016-09-16T14:00:08.03+0900 [APP/0]      OUT 05:00:08 php-fpm | [16-Sep-2016 05:00:08] NOTICE: fpm is running, pid 46
2016-09-16T14:00:08.03+0900 [APP/0]      OUT 05:00:08 php-fpm | [16-Sep-2016 05:00:08] NOTICE: ready to handle connections
2016-09-16T14:00:08.09+0900 [APP/0]      OUT 05:00:08 httpd   | [Fri Sep 16 05:00:08.075857 2016] [mpm_event:notice] [pid 44:tid 140622849943424] AH00489: Apache/2.4.23 (Unix) configured -- resuming normal operations
2016-09-16T14:00:08.09+0900 [APP/0]      OUT 05:00:08 httpd   | [Fri Sep 16 05:00:08.075970 2016] [mpm_event:info] [pid 44:tid 140622849943424] AH00490: Server built: Jul 18 2016 19:31:07
2016-09-16T14:00:08.09+0900 [APP/0]      OUT 05:00:08 httpd   | [Fri Sep 16 05:00:08.075987 2016] [core:notice] [pid 44:tid 140622849943424] AH00094: Command line: '/app/httpd/bin/httpd -f /home/vcap/app/httpd/conf/httpd.conf -D FOREGROUND'
2016-09-16T14:00:08.27+0900 [CELL/0]     OUT Container became healthy
2016-09-16T14:02:33.90+0900 [APP/0]      OUT 05:02:33 httpd   | 126.112.254.162 - - [16/Sep/2016:05:02:33 +0000] "GET / HTTP/1.1" 200 32 vcap_request_id=b248b33d-d780-4f5c-658e-27aed994c12e peer_addr=10.10.81.9
2016-09-16T14:02:33.91+0900 [RTR/3]      OUT hello-tmaki.cfapps.io - [16/09/2016:05:02:33.898 +0000] "GET / HTTP/1.1" 200 0 32 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36" 10.10.2.230:20108 x_forwarded_for:"126.112.254.162" x_forwarded_proto:"http" vcap_request_id:b248b33d-d780-4f5c-658e-27aed994c12e response_time:0.016536782 app_id:18f0966b-f3ba-4c5d-92e7-9714a1c40ac2 index:0

```

また`cf logs <App>`で今後流れるログを確認することができます(`tail -f`相当)。

### アプリケーションの削除

`cf delete`でアプリケーションを削除できます。

``` console
$ cf delete hello-tmaki

Really delete the app hello-tmaki?> y
Deleting app hello-tmaki in org tmaki / space development as ****@gmail.com...
OK
```

### `--random-route`を使う

先ほどはアプリケーション名に`-<your name>`をつけ一意にしました。`hello`だと重複する可能性が高いためです。実はアプリケーション名自体はスペース内で一意であればよく、一意にすべきはホスト名(`xxxx.cfapps.io`の`xxxx`の部分)です。これは`-n`または`--hostname`で指定できます。
一意なホスト名にするには`--random-route`を追加すれば良いです。

``` console
$ cf push hello -m 64m --random-route
```

`cf apps`を確認すると、ホスト名が`hello-unretained-agenesis`になっていることがわかります。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name    requested state   instances   memory   disk   urls   
hello   started           1/1         64M      1G     hello-unretained-agenesis.cfapps.io 
```

この場合、[http://hello-unretained-agenesis.cfapps.io](
http://hello-unretained-agenesis.cfapps.io)にアクセスできます。


### Buildpackを指定する

`cf push`でアプリケーションをアップロードした後、ステージングとよばれるフェーズでランタイム(JREやサーバーなど)を追加し実行可能なDropletという形式を作成します。Dropletを作るためのBuildpackとよばれる仕組みが使われます。

アップロードしたファイル群(アーティファクト)から自動で適用すべきBuildpackが判断され、Cloud Foundryに他言語対応はここで行われています。

利用可能なBuildpack一覧は`cf buildpacks`で取得できます。

``` console
$ cf buildpacks
Getting buildpacks...

buildpack                    position   enabled   locked   filename
staticfile_buildpack         1          true      true     staticfile_buildpack-cached-v1.3.10.zip
java_buildpack               2          true      true     java-buildpack-offline-v3.9.zip
ruby_buildpack               3          true      true     ruby_buildpack-cached-v1.6.24.zip
nodejs_buildpack             4          true      true     nodejs_buildpack-cached-v1.5.19.zip
go_buildpack                 5          true      true     go_buildpack-cached-v1.7.12.zip
python_buildpack             6          true      true     python_buildpack-cached-v1.5.9.zip
php_buildpack                7          true      true     php_buildpack-cached-v4.3.19.zip
liberty_buildpack            8          true      true     liberty_buildpack.zip
binary_buildpack             9          true      true     binary_buildpack-cached-v1.0.3.zip
dotnet_core_buildpack_beta   10         true      true     dotnet-core_buildpack-cached-v1.0.0.zip
```

デフォルトでは、`cf push`でアーティファクトをアップロードした後、利用可能なBuildpackを全てダウンロードし、優先順(`position`順)にチェックし、対象のBuildpackを特定しDroplet(実行可能な形式)を作成します。

今回の場合は、php_buildpackが検知され、PHP用のDropletが作成されます。

Buildpackは`-b`で明示的に指定できます。明示することで自動検出のための時間を短縮できます。

``` console
$ cf push hello -m 64m --random-route -b php_buildpack
Creating app hello in org APJ / space staging as tmaki@pivotal.io...
OK

Creating route hello-unretained-agenesis.cfapps.io...
OK

Binding hello-unretained-agenesis.cfapps.io to hello...
OK

Uploading hello...
Uploading app files from: /Users/makit/cfws/php/hello-cf
Uploading 3.9M, 4209 files
Done uploading               
OK

Starting app hello in org APJ / space staging as tmaki@pivotal.io...
Downloading php_buildpack...
Downloaded php_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (5.8M)
Staging...
Installing HTTPD
Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_httpd_httpd-2.4.23-linux-x64.tgz] to [/tmp]
WARNING: The version defined in `composer.json` will be used.
PHP 7.0.9
-------> Buildpack version 4.3.19
WARNING: A version of PHP has been specified in both `composer.json` and `./bp-config/options.json`.
Installing PHP
Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_php7_php7-7.0.9-linux-x64-1472493075.tgz] to [/tmp]
The extension 'tokenizer' is not provided by this buildpack.
The extension 'dom' is not provided by this buildpack.
The extension 'json' is not provided by this buildpack.
The extension 'pcre' is not provided by this buildpack.
The extension 'reflection' is not provided by this buildpack.
The extension 'spl' is not provided by this buildpack.
Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_php7_php7-7.0.9-linux-x64-1472493075.tgz] to [/tmp]
Downloaded [file:///tmp/buildpacks/476d17b65c586ec735d679d708da80bf/dependencies/https___buildpacks.cloudfoundry.org_php_binaries_trusty_composer_1.2.0_composer.phar] to [/tmp]
Loading composer repositories with package information
Installing dependencies from lock file
  - Installing doctrine/inflector (v1.1.0)
    Downloading
  - Installing symfony/polyfill-mbstring (v1.2.0)
    Downloading
  - Installing symfony/console (v3.0.9)
    Downloading
  - Installing symfony/translation (v3.0.9)
    Downloading
  - Installing nesbot/carbon (1.21.0)
    Downloading
  - Installing paragonie/random_compat (v1.4.1)
    Downloading
  - Installing illuminate/contracts (v5.2.45)
    Downloading
  - Installing illuminate/support (v5.2.45)
    Downloading
  - Installing illuminate/console (v5.2.45)
    Downloading
  - Installing symfony/http-foundation (v3.0.9)
    Downloading
  - Installing symfony/finder (v3.0.9)
    Downloading
  - Installing illuminate/session (v5.2.45)
    Downloading
  - Installing symfony/polyfill-util (v1.2.0)
    Downloading
  - Installing symfony/polyfill-php56 (v1.2.0)
    Downloading
  - Installing symfony/event-dispatcher (v3.1.4)
    Downloading
  - Installing psr/log (1.0.0)
    Downloading
  - Installing symfony/debug (v3.0.9)
    Downloading
  - Installing symfony/http-kernel (v3.0.9)
    Downloading
  - Installing nikic/fast-route (v0.7.0)
    Downloading
  - Installing mtdowling/cron-expression (v1.1.0)
    Downloading
  - Installing monolog/monolog (1.21.0)
    Downloading
    Downloading
  - Installing illuminate/container (v5.2.45)
    Downloading
  - Installing illuminate/events (v5.2.45)
    Downloading
  - Installing illuminate/view (v5.2.45)
    Downloading
  - Installing illuminate/filesystem (v5.2.45)
    Downloading
  - Installing illuminate/translation (v5.2.45)
    Downloading
  - Installing illuminate/validation (v5.2.45)
  - Installing symfony/process (v3.0.9)
    Downloading
  - Installing illuminate/queue (v5.2.45)
    Downloading
    Downloading
  - Installing illuminate/pipeline (v5.2.45)
  - Installing illuminate/pagination (v5.2.45)
    Downloading
  - Installing illuminate/http (v5.2.45)
  - Installing illuminate/hashing (v5.2.45)
    Downloading
    Downloading
  - Installing illuminate/config (v5.2.45)
  - Installing illuminate/cache (v5.2.45)
    Downloading
    Downloading
  - Installing illuminate/broadcasting (v5.2.45)
  - Installing illuminate/bus (v5.2.45)
    Downloading
  - Installing illuminate/auth (v5.2.45)
    Downloading
  - Installing laravel/lumen-framework (v5.2.9)
    Downloading
  - Installing vlucas/phpdotenv (v2.4.0)
    Downloading
Generating autoload files
Finished: [2016-09-16 05:05:34.766784]
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (1.5M)
Uploaded droplet (62.4M)
Uploading complete
Destroying container
Successfully destroyed container

1 of 1 instances running

App started


OK

App hello was started using this command `$HOME/.bp/bin/start`

Showing health and status for app hello in org APJ / space staging as tmaki@pivotal.io...
OK

requested state: started
instances: 1/1
usage: 64M x 1 instances
urls: hello-unretained-agenesis.cfapps.io
last uploaded: Fri Sep 16 05:05:19 UTC 2016
stack: cflinuxfs2
buildpack: php_buildpack

     state     since                    cpu    memory         disk           details
#0   running   2016-09-16 02:06:01 PM   0.0%   28.4M of 64M   208.6M of 1G
```

### Manifestファイルを作成

ここまで`cf`コマンドで指定してきたオプションは`manifest.yml`というyamlファイルに定義できます。

`cf push hello -m 64m --random-route -b php_buildpack`を`manifest.yml`で表すと、

``` yaml
---
applications:
  - name: hello
    memory: 64m
    buildpack: php_buildpack
    random-route: true
```

となります。

このManifestファイルがあれば実行コマンドは`cf push`だけで良いです。

``` console
$ cf push
Using manifest file /Users/makit/cfws/php/manifest.yml
(以下、略)
```

