## 簡単なアプリケーションをデプロイ (PHP編)

PHPのとても簡単なWebアプリケーションを作成 & デプロイしましょう。

### プロジェクトの作成

`hello-cf`フォルダを作成してください。

``` console
$ mkdir hello-cf
$ cd hello-cf
```

`index.php`をさくせうして、次の内容を記述してください。

``` php
<?php
echo "Hello World!";
```

### アプリケーションをCloud FoundryにPush

作成したアプリケーションをCloud FoundryにPushしましょう。

`cf`コマンドを使ってCloud Foundryにアプリケーションをデプロイできます。

``` console
$ cf push hello-<your name> -m 32m
```

`<your name>`は自分の名前などを置換して、一意にしてください。以下では`<your name>`を`tmaki`とします。適宜自分の名前に読み替えてください。
`-m`で使用するメモリ量を指定します。Pivotal Web Servicesではデフォルトで1GBが使用されますが、今回のPHPアプリではそんなに必要ないのでここでは32MBを指定しています。

``` console
$ cf push hello-tmaki -m 32m
Creating app hello-tmaki in org APJ / space Development as tmaki@pivotal.io...
OK

Creating route hello-tmaki.cfapps.io...
OK

Binding hello-tmaki.cfapps.io to hello-tmaki...
OK

Uploading hello-tmaki...
Uploading app files from: /Users/makit/hello-cf
Uploading 164B, 1 files
Done uploading               
OK

Starting app hello-tmaki in org APJ / space Development as tmaki@pivotal.io...
Downloading binary_buildpack...
Downloading dotnet_core_buildpack...
Downloading staticfile_buildpack...
Downloading java_buildpack...
Downloading dotnet_core_buildpack_beta...
Downloaded staticfile_buildpack
Downloading nodejs_buildpack...
Downloading ruby_buildpack...
Downloaded dotnet_core_buildpack
Downloading php_buildpack...
Downloaded dotnet_core_buildpack_beta
Downloading go_buildpack...
Downloaded binary_buildpack
Downloaded java_buildpack
Downloading python_buildpack...
Downloaded nodejs_buildpack
Downloaded ruby_buildpack
Downloaded php_buildpack
Downloaded go_buildpack
Downloaded python_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (194B)
Staging...
-------> Buildpack version 4.3.25
Installing HTTPD
HTTPD 2.4.25
Downloaded [file:///tmp/buildpacks/6262a5d0bdc6c1824409ca1df384dfa6/dependencies/https___buildpacks.cloudfoundry.org_dependencies_httpd_httpd-2.4.25-linux-x64.tgz] to [/tmp]
Installing PHP
PHP 5.5.38
Downloaded [file:///tmp/buildpacks/6262a5d0bdc6c1824409ca1df384dfa6/dependencies/https___buildpacks.cloudfoundry.org_dependencies_php_php-5.5.38-linux-x64-1479852231.tgz] to [/tmp]
Finished: [2017-01-25 09:42:23.027861]
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (199B)
Uploaded droplet (53M)
Uploading complete
Destroying container
Successfully destroyed container

0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App hello-tmaki was started using this command `$HOME/.bp/bin/start`

Showing health and status for app hello-tmaki in org APJ / space Development as tmaki@pivotal.io...
OK

requested state: started
instances: 1/1
usage: 32M x 1 instances
urls: hello-tmaki.cfapps.io
last uploaded: Wed Jan 25 09:42:05 UTC 2017
stack: cflinuxfs2
buildpack: php 4.3.25

     state     since                    cpu    memory         disk         details
#0   running   2017-01-25 06:42:48 PM   0.0%   22.5M of 32M   156M of 1G
````

これでデプロイに成功しました。
`cf apps`でデプロイされているアプリケーションの一覧を取得できます。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name          requested state   instances   memory   disk   urls   
hello-tmaki   started           1/1         32M      1G     hello-tmaki.cfapps.io <---
```

`urls`の列に出力されている値がアプリケーションのURLです。この場合は[https://hello-tmaki.cfapps.io](https://hello-tmaki.cfapps.io)です。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/f2519229-95d1-4a05-1f78-f471e124a178.png)


Cloud Foundry上にデプロイされたアプリケーションにもアクセスできました。

[PWSの管理画面](https://console.run.pivotal.io)を見てみましょう。(**スクリーンショットの画像は古いです**)
![image](https://qiita-image-store.s3.amazonaws.com/0/1852/4fd1565a-9bad-63a2-9382-6e7f59363b10.png)

「Space development」をクリックしてください。`development`というスペースにデプロイされているアプリケーションの一覧を確認できます。`hello-<your name>`が表示されています。(**スクリーンショットの画像は古いです**)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/a0684f30-3757-2cfe-0580-c19b3598a499.png)

`hello-<your name>`をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/90829563-81ed-bd81-07ab-35ca8e957e3f.png)


アプリケーションの情報が確認できます。

直近のログは`cf logs <App> --recent`で確認できます。

``` console
$ cf logs hello-tmaki --recent
Connected, dumping recent logs for app hello-tmaki in org APJ / space Development as tmaki@pivotal.io...

2017-01-25T18:41:59.05+0900 [API/7]      OUT Created app with guid 1bdebcd0-e997-4475-81f4-c6ffc41306a9
2017-01-25T18:42:03.81+0900 [API/6]      OUT Updated app with guid 1bdebcd0-e997-4475-81f4-c6ffc41306a9 ({"route"=>"d00a9541-16ea-4c06-bd8a-e3a5625d95a5", :verb=>"add", :relation=>"routes", :related_guid=>"d00a9541-16ea-4c06-bd8a-e3a5625d95a5"})
2017-01-25T18:42:15.87+0900 [API/7]      OUT Updated app with guid 1bdebcd0-e997-4475-81f4-c6ffc41306a9 ({"state"=>"STARTED"})
2017-01-25T18:42:16.26+0900 [STG/0]      OUT Downloading binary_buildpack...
2017-01-25T18:42:16.26+0900 [STG/0]      OUT Downloading dotnet_core_buildpack...
2017-01-25T18:42:16.26+0900 [STG/0]      OUT Downloading staticfile_buildpack...
2017-01-25T18:42:16.26+0900 [STG/0]      OUT Downloading java_buildpack...
2017-01-25T18:42:16.27+0900 [STG/0]      OUT Downloading dotnet_core_buildpack_beta...
2017-01-25T18:42:16.31+0900 [STG/0]      OUT Downloaded staticfile_buildpack
2017-01-25T18:42:16.31+0900 [STG/0]      OUT Downloading nodejs_buildpack...
2017-01-25T18:42:16.31+0900 [STG/0]      OUT Downloaded binary_buildpack
2017-01-25T18:42:16.31+0900 [STG/0]      OUT Downloading ruby_buildpack...
2017-01-25T18:42:16.32+0900 [STG/0]      OUT Downloaded dotnet_core_buildpack
2017-01-25T18:42:16.32+0900 [STG/0]      OUT Downloading php_buildpack...
2017-01-25T18:42:16.32+0900 [STG/0]      OUT Downloaded dotnet_core_buildpack_beta
2017-01-25T18:42:16.32+0900 [STG/0]      OUT Downloading go_buildpack...
2017-01-25T18:42:16.34+0900 [STG/0]      OUT Downloaded java_buildpack
2017-01-25T18:42:16.34+0900 [STG/0]      OUT Downloading python_buildpack...
2017-01-25T18:42:16.35+0900 [STG/0]      OUT Downloaded nodejs_buildpack
2017-01-25T18:42:16.36+0900 [STG/0]      OUT Downloaded ruby_buildpack
2017-01-25T18:42:16.36+0900 [STG/0]      OUT Downloaded php_buildpack
2017-01-25T18:42:16.37+0900 [STG/0]      OUT Downloaded go_buildpack
2017-01-25T18:42:16.38+0900 [STG/0]      OUT Downloaded python_buildpack
2017-01-25T18:42:16.38+0900 [STG/0]      OUT Creating container
2017-01-25T18:42:18.75+0900 [STG/0]      OUT Successfully created container
2017-01-25T18:42:18.75+0900 [STG/0]      OUT Downloading app package...
2017-01-25T18:42:18.84+0900 [STG/0]      OUT Downloaded app package (194B)
2017-01-25T18:42:18.84+0900 [STG/0]      OUT Staging...
2017-01-25T18:42:20.23+0900 [STG/0]      OUT -------> Buildpack version 4.3.25
2017-01-25T18:42:20.30+0900 [STG/0]      OUT Installing HTTPD
2017-01-25T18:42:20.30+0900 [STG/0]      OUT HTTPD 2.4.25
2017-01-25T18:42:20.49+0900 [STG/0]      OUT Downloaded [file:///tmp/buildpacks/6262a5d0bdc6c1824409ca1df384dfa6/dependencies/https___buildpacks.cloudfoundry.org_dependencies_httpd_httpd-2.4.25-linux-x64.tgz] to [/tmp]
2017-01-25T18:42:20.78+0900 [STG/0]      OUT Installing PHP
2017-01-25T18:42:20.78+0900 [STG/0]      OUT PHP 5.5.38
2017-01-25T18:42:21.25+0900 [STG/0]      OUT Downloaded [file:///tmp/buildpacks/6262a5d0bdc6c1824409ca1df384dfa6/dependencies/https___buildpacks.cloudfoundry.org_dependencies_php_php-5.5.38-linux-x64-1479852231.tgz] to [/tmp]
2017-01-25T18:42:23.02+0900 [STG/0]      OUT Finished: [2017-01-25 09:42:23.027861]
2017-01-25T18:42:32.76+0900 [STG/0]      OUT Exit status 0
2017-01-25T18:42:32.76+0900 [STG/0]      OUT Staging complete
2017-01-25T18:42:32.77+0900 [STG/0]      OUT Uploading droplet, build artifacts cache...
2017-01-25T18:42:32.77+0900 [STG/0]      OUT Uploading build artifacts cache...
2017-01-25T18:42:32.77+0900 [STG/0]      OUT Uploading droplet...
2017-01-25T18:42:32.87+0900 [STG/0]      OUT Uploaded build artifacts cache (199B)
2017-01-25T18:42:40.17+0900 [STG/0]      OUT Uploaded droplet (53M)
2017-01-25T18:42:40.18+0900 [STG/0]      OUT Uploading complete
2017-01-25T18:42:40.23+0900 [STG/0]      OUT Destroying container
2017-01-25T18:42:40.82+0900 [CELL/0]     OUT Creating container
2017-01-25T18:42:42.42+0900 [STG/0]      OUT Successfully destroyed container
2017-01-25T18:42:42.80+0900 [CELL/0]     OUT Successfully created container
2017-01-25T18:42:46.64+0900 [CELL/0]     OUT Starting health monitoring of container
2017-01-25T18:42:47.12+0900 [APP/PROC/WEB/0]OUT 09:42:47 php-fpm | [25-Jan-2017 09:42:47] NOTICE: fpm is running, pid 48
2017-01-25T18:42:47.12+0900 [APP/PROC/WEB/0]OUT 09:42:47 php-fpm | [25-Jan-2017 09:42:47] NOTICE: ready to handle connections
2017-01-25T18:42:47.18+0900 [APP/PROC/WEB/0]OUT 09:42:47 httpd   | [Wed Jan 25 09:42:47.173921 2017] [mpm_event:notice] [pid 46:tid 140141928654720] AH00489: Apache/2.4.25 (Unix) configured -- resuming normal operations
2017-01-25T18:42:47.18+0900 [APP/PROC/WEB/0]OUT 09:42:47 httpd   | [Wed Jan 25 09:42:47.174063 2017] [mpm_event:info] [pid 46:tid 140141928654720] AH00490: Server built: Dec 20 2016 22:27:46
2017-01-25T18:42:47.18+0900 [APP/PROC/WEB/0]OUT 09:42:47 httpd   | [Wed Jan 25 09:42:47.174081 2017] [core:notice] [pid 46:tid 140141928654720] AH00094: Command line: '/app/httpd/bin/httpd -f /home/vcap/app/httpd/conf/httpd.conf -D FOREGROUND'
2017-01-25T18:42:48.73+0900 [CELL/0]     OUT Container became healthy
2017-01-25T18:44:03.71+0900 [APP/PROC/WEB/0]OUT 09:44:03 httpd   | 202.241.169.198 - - [25/Jan/2017:09:44:03 +0000] "GET / HTTP/1.1" 200 32 vcap_request_id=ffcbfcff-bed1-48d8-4f97-537e5d1b01ef peer_addr=10.10.81.6
2017-01-25T18:44:03.72+0900 [RTR/2]      OUT hello-tmaki.cfapps.io - [2017-01-25T09:44:03.717+0000] "GET / HTTP/1.1" 200 0 32 "http://qiita.com/drafts/8ee9714204c6fc593023/edit" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.95 Safari/537.36" "10.10.2.26:58469" "10.10.147.72:60015" x_forwarded_for:"202.241.169.198" x_forwarded_proto:"http" vcap_request_id:"ffcbfcff-bed1-48d8-4f97-537e5d1b01ef" response_time:0.003913261 app_id:"1bdebcd0-e997-4475-81f4-c6ffc41306a9" app_index:"0" x_b3_traceid:"fc9789636e447329" x_b3_spanid:"fc9789636e447329" x_b3_parentspanid:"-"
2017-01-25T18:44:04.11+0900 [APP/PROC/WEB/0]OUT 09:44:04 httpd   | [Wed Jan 25 09:44:04.109168 2017] [core:info] [pid 59:tid 140141712611072] [client 202.241.169.198:1158] AH00128: File does not exist: /home/vcap/app/htdocs/favicon.ico, referer: http://hello-tmaki.cfapps.io/
2017-01-25T18:44:04.11+0900 [APP/PROC/WEB/0]OUT 09:44:04 httpd   | 202.241.169.198 - - [25/Jan/2017:09:44:04 +0000] "GET /favicon.ico HTTP/1.1" 404 209 vcap_request_id=375f79ab-0a7d-42fb-555b-5b9256e12bb4 peer_addr=10.10.81.6
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

> **Note**
> `cf delete`に`-r`オプションをつけると、ルーティング情報も合わせて削除されます。`-f`オプションをつけると、確認を聞かれません。

### `--random-route`を使う

先ほどはアプリケーション名に`-<your name>`をつけ一意にしました。`hello`だと重複する可能性が高いためです。実はアプリケーション名自体はスペース内で一意であればよく、一意にすべきはホスト名(`xxxx.cfapps.io`の`xxxx`の部分)です。これは`-n`または`--hostname`で指定できます。
一意なホスト名にするには`--random-route`を追加すれば良いです。

``` console
$ cf push hello -m 32m --random-route
```

`cf apps`を確認すると、ホスト名が`hello-unretained-agenesis`になっていることがわかります。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name    requested state   instances   memory   disk   urls   
hello   started           1/1         32M      1G     hello-unretained-agenesis.cfapps.io 
```

この場合、[https://hello-unretained-agenesis.cfapps.io](
https://hello-unretained-agenesis.cfapps.io)にアクセスできます。


### Buildpackを指定する

`cf push`でアプリケーションをアップロードした後、ステージングとよばれるフェーズでランタイム(JREやサーバーなど)を追加し実行可能なDropletという形式を作成します。Dropletを作るためのBuildpackとよばれる仕組みが使われます。

アップロードしたファイル群(アーティファクト)から自動で適用すべきBuildpackが判断され、Cloud Foundryに他言語対応はここで行われています。

利用可能なBuildpack一覧は`cf buildpacks`で取得できます。

``` console
$ cf buildpacks
Getting buildpacks...

buildpack                    position   enabled   locked   filename
staticfile_buildpack         1          true      true     staticfile_buildpack-cached-v1.3.16.zip
java_buildpack               2          true      true     java-buildpack-offline-v3.12.zip
ruby_buildpack               3          true      true     ruby_buildpack-cached-v1.6.33.zip
nodejs_buildpack             4          true      true     nodejs_buildpack-cached-v1.5.27.zip
go_buildpack                 5          true      true     go_buildpack-cached-v1.7.17.zip
python_buildpack             6          true      true     python_buildpack-cached-v1.5.14.zip
php_buildpack                7          true      true     php_buildpack-cached-v4.3.25.zip
dotnet_core_buildpack        8          true      true     dotnet-core_buildpack-cached-v1.0.10.zip
dotnet_core_buildpack_beta   9          true      false    dotnet-core_buildpack-cached-v1.0.0.zip
binary_buildpack             10         true      true     binary_buildpack-cached-v1.0.7.zip
```

デフォルトでは、`cf push`でアーティファクトをアップロードした後、利用可能なBuildpackを全てダウンロードし、優先順(`position`順)にチェックし、対象のBuildpackを特定しDroplet(実行可能な形式)を作成します。

今回の場合は、php_buildpackが検知され、PHP用のDropletが作成されます。

Buildpackは`-b`で明示的に指定できます。明示することで自動検出のための時間を短縮できます。

``` console
$ cf push hello -m 32m --random-route -b php_buildpack
Creating app hello in org APJ / space Development as tmaki@pivotal.io...
OK

Creating route hello-toponymical-hordein.cfapps.io...
OK

Binding hello-toponymical-hordein.cfapps.io to hello...
OK

Uploading hello...
Uploading app files from: /Users/makit/hello-cf
Uploading 164B, 1 files
Done uploading               
OK

Starting app hello in org APJ / space Development as tmaki@pivotal.io...
Downloading php_buildpack...
Downloaded php_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (194B)
Staging...
-------> Buildpack version 4.3.25
Installing HTTPD
HTTPD 2.4.25
Downloaded [file:///tmp/buildpacks/6262a5d0bdc6c1824409ca1df384dfa6/dependencies/https___buildpacks.cloudfoundry.org_dependencies_httpd_httpd-2.4.25-linux-x64.tgz] to [/tmp]
Installing PHP
PHP 5.5.38
Downloaded [file:///tmp/buildpacks/6262a5d0bdc6c1824409ca1df384dfa6/dependencies/https___buildpacks.cloudfoundry.org_dependencies_php_php-5.5.38-linux-x64-1479852231.tgz] to [/tmp]
Finished: [2017-01-25 09:59:05.060579]
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (198B)
Uploaded droplet (53M)
Uploading complete
Destroying container
Successfully destroyed container

1 of 1 instances running

App started


OK

App hello was started using this command `$HOME/.bp/bin/start`

Showing health and status for app hello in org APJ / space Development as tmaki@pivotal.io...
OK

requested state: started
instances: 1/1
usage: 32M x 1 instances
urls: hello-toponymical-hordein.cfapps.io
last uploaded: Wed Jan 25 09:58:56 UTC 2017
stack: cflinuxfs2
buildpack: php_buildpack

     state     since                    cpu    memory         disk         details
#0   running   2017-01-25 06:59:31 PM   0.0%   16.4M of 32M   156M of 1G
```

### Manifestファイルを作成

ここまで`cf`コマンドで指定してきたオプションは`manifest.yml`というyamlファイルに定義できます。

`cf push hello -m 64m --random-route -b php_buildpack`を`manifest.yml`で表すと、

``` yaml
---
applications:
  - name: hello
    memory: 32m
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
