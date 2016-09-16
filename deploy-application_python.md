## 簡単なアプリケーションをデプロイ (Python編)

Flaskを使ってとても簡単なWebアプリケーションを作成 & デプロイしましょう。

### プロジェクトの作成

`hello-cf`フォルダを作成してください。

``` console
$ mkdir hello-cf
$ cd hello-cf
```

`hello.py`を作成して、次の内容を記述してください。

``` python
from flask import Flask
import os

app = Flask(__name__)

port = int(os.getenv("PORT"))

@app.route('/')
def hello():
    return 'Hello World!'

if __name__ == '__main__':
    app.run(host = '0.0.0.0', port = port)
```

次のように`requirements.txt`に依存ライブラリ(Flask)を定義します。

```
Flask
```

まずはローカルでアプリケーションを実行してみましょう。

``` console
$ sudo pip install -r requirements.txt
$ PORT=8080 python hello.py 
```

[http://localhost:8080](http://localhost:8080)にアクセスしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/02020313-0016-ba4e-e2bd-f4ddd5db40f0.png)

Hello World!が表示されれば成功です。

### アプリケーションをCloud FoundryにPush

ビルドしたアプリケーションをCloud FoundryにPushしましょう。

まずはCloud Foundry上で使用するPythonのバージョンを`runtime.txt`に記述します。今回はPython 3.5.2を使用します。

```
python-3.5.2
```

> **ノート: 指定できるPythonのバージョン**
> 
> 指定できるPythonのバージョンは使用するBuildpackによります。
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
> python_buildpack             6          true      true     python_buildpack-cached-v1.5.9.zip <-----
> php_buildpack                7          true      true     php_buildpack-cached-v4.3.19.zip
> liberty_buildpack            8          true      true     liberty_buildpack.zip
> binary_buildpack             9          true      true     binary_buildpack-cached-v1.0.3.zip
> dotnet_core_buildpack_beta   10         true      true     dotnet-core_buildpack-cached-v1.0.0.zip
> ```
> 
> 上の例ではPythonのBuildpackバージョンは1.5.9です。BuildpackのリリースページにそのBuildpackバージョンで利用出来るPythonのバージョンが列挙されています。
> 
> https://github.com/cloudfoundry/python-buildpack/releases/tag/v1.5.9

次に、`Procfile`に実行するコマンドを記述します(Heroku互換です)。

```
web: python hello.py
```

フォルダ構成は次にようになっているでしょうか。

``` console
$ ls -la
total 32
drwxr-xr-x  6 makit  720748206  204  9 16 11:09 .
drwxr-xr-x  6 makit  720748206  204  9 16 11:00 ..
-rw-r--r--  1 makit  720748206   20  9 16 11:04 Procfile
-rw-r--r--  1 makit  720748206  215  9 16 11:09 hello.py
-rw-r--r--  1 makit  720748206    6  9 16 11:04 requirements.txt
-rw-r--r--  1 makit  720748206   13  9 16 11:07 runtime.txt
```

この状態で`cf`コマンドを使ってCloud Foundryにアプリケーションをデプロイできます。

``` console
$ cf push hello-<your name> -m 32m
```

`<your name>`は自分の名前などを置換して、一意にしてください。以下では`<your name>`を`tmaki`とします。適宜自分の名前に読み替えてください。
`-m`で使用するメモリ量を指定します。Pivotal Web Servicesではデフォルトで1GBが使用されますが、Pythonアプリではそんなに必要ないのでここでは32MBを指定しています。

``` console
$ cf push hello-tmaki -m 32m
Creating app hello-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK

Creating route hello-tmaki.cfapps.io...
OK

Binding hello-tmaki.cfapps.io to hello-tmaki...
OK

Uploading hello-tmaki...
Uploading app files from: /Users/makit/cfws/python
Uploading 694B, 4 files
Done uploading               
OK

Starting app hello-tmaki in org APJ / space staging as tmaki@pivotal.io...
Downloading dotnet_core_buildpack_beta...
Downloading nodejs_buildpack...
Downloading java_buildpack...
Downloading ruby_buildpack...
Downloading staticfile_buildpack...
Downloaded java_buildpack
Downloaded nodejs_buildpack
Downloading liberty_buildpack...
Downloading go_buildpack...
Downloaded dotnet_core_buildpack_beta
Downloaded staticfile_buildpack
Downloading python_buildpack...
Downloading binary_buildpack...
Downloaded liberty_buildpack
Downloading php_buildpack...
Downloaded python_buildpack
Downloaded go_buildpack
Downloaded binary_buildpack
Downloaded php_buildpack
Downloaded ruby_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (815B)
Staging...
-------> Buildpack version 1.5.9
-----> Installing python-3.5.2
Downloaded [file:///tmp/buildpacks/77e9434bd8c93e72ac62bfcf4e2da90c/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_python_python-3.5.2-linux-x64.tgz]
     $ pip install -r requirements.txt
       Collecting Flask (from -r requirements.txt (line 1))
         Downloading Flask-0.11.1-py2.py3-none-any.whl (80kB)
       Collecting Werkzeug>=0.7 (from Flask->-r requirements.txt (line 1))
         Downloading Werkzeug-0.11.11-py2.py3-none-any.whl (306kB)
       Collecting itsdangerous>=0.21 (from Flask->-r requirements.txt (line 1))
         Downloading itsdangerous-0.24.tar.gz (46kB)
       Collecting Jinja2>=2.4 (from Flask->-r requirements.txt (line 1))
         Downloading Jinja2-2.8-py2.py3-none-any.whl (263kB)
       Collecting click>=2.0 (from Flask->-r requirements.txt (line 1))
         Downloading click-6.6.tar.gz (283kB)
       Collecting MarkupSafe (from Jinja2>=2.4->Flask->-r requirements.txt (line 1))
         Downloading MarkupSafe-0.23.tar.gz
       Installing collected packages: Werkzeug, itsdangerous, MarkupSafe, Jinja2, click, Flask
         Running setup.py install for itsdangerous: started
           Running setup.py install for itsdangerous: finished with status 'done'
         Running setup.py install for MarkupSafe: started
           Running setup.py install for MarkupSafe: finished with status 'done'
         Running setup.py install for click: started
           Running setup.py install for click: finished with status 'done'
       Successfully installed Flask-0.11.1 Jinja2-2.8 MarkupSafe-0.23 Werkzeug-0.11.11 click-6.6 itsdangerous-0.24
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (43.5M)
Uploaded droplet (43.5M)
Uploading complete
Destroying container
Successfully destroyed container

1 of 1 instances running

App started


OK

App hello-tmaki was started using this command `python hello.py`

Showing health and status for app hello-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK

requested state: started
instances: 1/1
usage: 32M x 1 instances
urls: hello-tmaki.cfapps.io
last uploaded: Fri Sep 16 02:30:25 UTC 2016
stack: cflinuxfs2
buildpack: python 1.5.9

     state     since                    cpu    memory        disk         details
#0   running   2016-09-16 11:31:26 AM   0.0%   1.7M of 32M   1.3M of 1G
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

2016-09-16T11:30:18.09+0900 [API/1]      OUT Created app with guid 0cbfe9c0-d315-4766-938c-9347d3559a79
2016-09-16T11:30:22.35+0900 [API/6]      OUT Updated app with guid 0cbfe9c0-d315-4766-938c-9347d3559a79 ({"route"=>"45a8104c-0b0a-474f-bbd8-1ed974005dd7", :verb=>"add", :relation=>:routes, :related_guid=>"45a8104c-0b0a-474f-bbd8-1ed974005dd7"})
2016-09-16T11:30:36.44+0900 [API/5]      OUT Updated app with guid 0cbfe9c0-d315-4766-938c-9347d3559a79 ({"state"=>"STARTED"})
2016-09-16T11:30:36.89+0900 [STG/0]      OUT Downloading dotnet_core_buildpack_beta...
2016-09-16T11:30:36.89+0900 [STG/0]      OUT Downloading nodejs_buildpack...
2016-09-16T11:30:36.89+0900 [STG/0]      OUT Downloading java_buildpack...
2016-09-16T11:30:36.89+0900 [STG/0]      OUT Downloading ruby_buildpack...
2016-09-16T11:30:36.89+0900 [STG/0]      OUT Downloading staticfile_buildpack...
2016-09-16T11:30:36.95+0900 [STG/0]      OUT Downloaded java_buildpack
2016-09-16T11:30:36.95+0900 [STG/0]      OUT Downloaded nodejs_buildpack
2016-09-16T11:30:36.95+0900 [STG/0]      OUT Downloading liberty_buildpack...
2016-09-16T11:30:36.95+0900 [STG/0]      OUT Downloading go_buildpack...
2016-09-16T11:30:36.95+0900 [STG/0]      OUT Downloaded dotnet_core_buildpack_beta
2016-09-16T11:30:36.95+0900 [STG/0]      OUT Downloaded staticfile_buildpack
2016-09-16T11:30:36.95+0900 [STG/0]      OUT Downloading python_buildpack...
2016-09-16T11:30:36.95+0900 [STG/0]      OUT Downloading binary_buildpack...
2016-09-16T11:30:36.99+0900 [STG/0]      OUT Downloaded liberty_buildpack
2016-09-16T11:30:36.99+0900 [STG/0]      OUT Downloading php_buildpack...
2016-09-16T11:30:36.99+0900 [STG/0]      OUT Downloaded python_buildpack
2016-09-16T11:30:36.99+0900 [STG/0]      OUT Downloaded go_buildpack
2016-09-16T11:30:37.00+0900 [STG/0]      OUT Downloaded binary_buildpack
2016-09-16T11:30:37.00+0900 [STG/0]      OUT Downloaded php_buildpack
2016-09-16T11:30:37.04+0900 [STG/0]      OUT Downloaded ruby_buildpack
2016-09-16T11:30:37.04+0900 [STG/0]      OUT Creating container
2016-09-16T11:30:38.06+0900 [STG/0]      OUT Successfully created container
2016-09-16T11:30:38.06+0900 [STG/0]      OUT Downloading app package...
2016-09-16T11:30:38.42+0900 [STG/0]      OUT Downloaded app package (815B)
2016-09-16T11:30:38.42+0900 [STG/0]      OUT Staging...
2016-09-16T11:30:39.31+0900 [STG/0]      OUT -------> Buildpack version 1.5.9
2016-09-16T11:30:39.40+0900 [STG/0]      OUT -----> Installing python-3.5.2
2016-09-16T11:30:40.75+0900 [STG/0]      OUT Downloaded [file:///tmp/buildpacks/77e9434bd8c93e72ac62bfcf4e2da90c/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_python_python-3.5.2-linux-x64.tgz]
2016-09-16T11:30:47.27+0900 [STG/0]      OUT      $ pip install -r requirements.txt
2016-09-16T11:30:47.82+0900 [STG/0]      OUT        Collecting Flask (from -r requirements.txt (line 1))
2016-09-16T11:30:47.92+0900 [STG/0]      OUT          Downloading Flask-0.11.1-py2.py3-none-any.whl (80kB)
2016-09-16T11:30:47.97+0900 [STG/0]      OUT        Collecting Werkzeug>=0.7 (from Flask->-r requirements.txt (line 1))
2016-09-16T11:30:48.02+0900 [STG/0]      OUT          Downloading Werkzeug-0.11.11-py2.py3-none-any.whl (306kB)
2016-09-16T11:30:48.09+0900 [STG/0]      OUT        Collecting itsdangerous>=0.21 (from Flask->-r requirements.txt (line 1))
2016-09-16T11:30:48.13+0900 [STG/0]      OUT          Downloading itsdangerous-0.24.tar.gz (46kB)
2016-09-16T11:30:48.63+0900 [STG/0]      OUT        Collecting Jinja2>=2.4 (from Flask->-r requirements.txt (line 1))
2016-09-16T11:30:48.67+0900 [STG/0]      OUT          Downloading Jinja2-2.8-py2.py3-none-any.whl (263kB)
2016-09-16T11:30:48.72+0900 [STG/0]      OUT        Collecting click>=2.0 (from Flask->-r requirements.txt (line 1))
2016-09-16T11:30:48.77+0900 [STG/0]      OUT          Downloading click-6.6.tar.gz (283kB)
2016-09-16T11:30:49.34+0900 [STG/0]      OUT        Collecting MarkupSafe (from Jinja2>=2.4->Flask->-r requirements.txt (line 1))
2016-09-16T11:30:49.37+0900 [STG/0]      OUT          Downloading MarkupSafe-0.23.tar.gz
2016-09-16T11:30:49.85+0900 [STG/0]      OUT        Installing collected packages: Werkzeug, itsdangerous, MarkupSafe, Jinja2, click, Flask
2016-09-16T11:30:50.09+0900 [STG/0]      OUT          Running setup.py install for itsdangerous: started
2016-09-16T11:30:50.68+0900 [STG/0]      OUT            Running setup.py install for itsdangerous: finished with status 'done'
2016-09-16T11:30:50.69+0900 [STG/0]      OUT          Running setup.py install for MarkupSafe: started
2016-09-16T11:30:51.48+0900 [STG/0]      OUT            Running setup.py install for MarkupSafe: finished with status 'done'
2016-09-16T11:30:51.59+0900 [STG/0]      OUT          Running setup.py install for click: started
2016-09-16T11:30:52.25+0900 [STG/0]      OUT            Running setup.py install for click: finished with status 'done'
2016-09-16T11:30:52.38+0900 [STG/0]      OUT        Successfully installed Flask-0.11.1 Jinja2-2.8 MarkupSafe-0.23 Werkzeug-0.11.11 click-6.6 itsdangerous-0.24
2016-09-16T11:31:15.02+0900 [STG/0]      OUT Exit status 0
2016-09-16T11:31:15.02+0900 [STG/0]      OUT Staging complete
2016-09-16T11:31:15.02+0900 [STG/0]      OUT Uploading droplet, build artifacts cache...
2016-09-16T11:31:15.02+0900 [STG/0]      OUT Uploading build artifacts cache...
2016-09-16T11:31:15.02+0900 [STG/0]      OUT Uploading droplet...
2016-09-16T11:31:15.87+0900 [STG/0]      OUT Uploaded build artifacts cache (43.5M)
2016-09-16T11:31:20.89+0900 [STG/0]      OUT Uploaded droplet (43.5M)
2016-09-16T11:31:20.90+0900 [STG/0]      OUT Uploading complete
2016-09-16T11:31:20.96+0900 [STG/0]      OUT Destroying container
2016-09-16T11:31:21.61+0900 [CELL/0]     OUT Creating container
2016-09-16T11:31:21.96+0900 [STG/0]      OUT Successfully destroyed container
2016-09-16T11:31:22.60+0900 [CELL/0]     OUT Successfully created container
2016-09-16T11:31:25.91+0900 [CELL/0]     OUT Starting health monitoring of container
2016-09-16T11:31:26.24+0900 [APP/0]      ERR  * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
2016-09-16T11:31:26.48+0900 [CELL/0]     OUT Container became healthy
2016-09-16T11:39:00.32+0900 [RTR/1]      OUT hello-tmaki.cfapps.io - [16/09/2016:02:39:00.318 +0000] "GET / HTTP/1.1" 200 0 12 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36" 10.10.66.73:25647 x_forwarded_for:"126.112.254.162" x_forwarded_proto:"http" vcap_request_id:630c90eb-2fc9-4d8d-5814-c23f76d6dcee response_time:0.002574044 app_id:0cbfe9c0-d315-4766-938c-9347d3559a79 index:0
2016-09-16T11:39:00.33+0900 [APP/0]      ERR 10.10.81.4 - - [16/Sep/2016 02:39:00] "GET / HTTP/1.1" 200 -
2016-09-16T11:39:00.74+0900 [RTR/4]      OUT hello-tmaki.cfapps.io - [16/09/2016:02:39:00.732 +0000] "GET /favicon.ico HTTP/1.1" 404 0 233 "http://hello-tmaki.cfapps.io/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36" 10.10.66.73:3339 x_forwarded_for:"126.112.254.162" x_forwarded_proto:"http" vcap_request_id:25164b6c-84cc-46ce-62bb-395fea16acfd response_time:0.012259777 app_id:0cbfe9c0-d315-4766-938c-9347d3559a79 index:0
2016-09-16T11:39:00.74+0900 [APP/0]      ERR 10.10.17.5 - - [16/Sep/2016 02:39:00] "GET /favicon.ico HTTP/1.1" 404 -

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
$ cf push hello -m 32m --random-route
```

`cf apps`を確認すると、ホスト名が`hello-mooned-falsification`になっていることがわかります。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name    requested state   instances   memory   disk   urls   
hello   started           1/1         32M      1G     hello-alphameric-laity.cfapps.io
```

この場合、[http://hello-alphameric-laity.cfapps.io](
http://hello-alphameric-laity.cfapps.io)にアクセスできます。


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

今回の場合は、python_buildpackが検知され、Python用のDropletが作成されます。

Buildpackは`-b`で明示的に指定できます。明示することで自動検出のための時間を短縮できます。

``` console
$ cf push hello -m 32m --random-route -b python_buildpack
Creating app hello in org APJ / space staging as tmaki@pivotal.io...
OK

Creating route hello-alphameric-laity.cfapps.io...
OK

Binding hello-alphameric-laity.cfapps.io to hello...
OK

Uploading hello...
Uploading app files from: /Users/makit/cfws/python
Uploading 694B, 4 files
Done uploading               
OK

Starting app hello in org APJ / space staging as tmaki@pivotal.io...
Downloaded python_buildpack
Creating container
Downloading python_buildpack...
Successfully created container
Downloading app package...
Downloaded app package (815B)
Staging...
-------> Buildpack version 1.5.9
-----> Installing python-3.5.2
Downloaded [file:///tmp/buildpacks/77e9434bd8c93e72ac62bfcf4e2da90c/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_python_python-3.5.2-linux-x64.tgz]
     $ pip install -r requirements.txt
       Collecting Flask (from -r requirements.txt (line 1))
         Downloading Flask-0.11.1-py2.py3-none-any.whl (80kB)
       Collecting itsdangerous>=0.21 (from Flask->-r requirements.txt (line 1))
         Downloading itsdangerous-0.24.tar.gz (46kB)
       Collecting Werkzeug>=0.7 (from Flask->-r requirements.txt (line 1))
         Downloading Werkzeug-0.11.11-py2.py3-none-any.whl (306kB)
       Collecting Jinja2>=2.4 (from Flask->-r requirements.txt (line 1))
         Downloading Jinja2-2.8-py2.py3-none-any.whl (263kB)
       Collecting click>=2.0 (from Flask->-r requirements.txt (line 1))
         Downloading click-6.6.tar.gz (283kB)
       Collecting MarkupSafe (from Jinja2>=2.4->Flask->-r requirements.txt (line 1))
         Downloading MarkupSafe-0.23.tar.gz
       Installing collected packages: itsdangerous, Werkzeug, MarkupSafe, Jinja2, click, Flask
         Running setup.py install for itsdangerous: started
           Running setup.py install for itsdangerous: finished with status 'done'
         Running setup.py install for MarkupSafe: started
           Running setup.py install for MarkupSafe: finished with status 'done'
         Running setup.py install for click: started
           Running setup.py install for click: finished with status 'done'
       Successfully installed Flask-0.11.1 Jinja2-2.8 MarkupSafe-0.23 Werkzeug-0.11.11 click-6.6 itsdangerous-0.24
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (43.5M)
Uploaded droplet (43.5M)
Uploading complete
Destroying container
Successfully destroyed container

1 of 1 instances running

App started


OK

App hello was started using this command `python hello.py`

Showing health and status for app hello in org APJ / space staging as tmaki@pivotal.io...
OK

requested state: started
instances: 1/1
usage: 32M x 1 instances
urls: hello-alphameric-laity.cfapps.io
last uploaded: Fri Sep 16 02:41:40 UTC 2016
stack: cflinuxfs2
buildpack: python_buildpack

     state     since                    cpu    memory        disk          details
#0   running   2016-09-16 11:42:46 AM   0.0%   1.1M of 32M   82.5M of 1G
```

### Manifestファイルを作成

ここまで`cf`コマンドで指定してきたオプションは`manifest.yml`というyamlファイルに定義できます。

`cf push hello -m 32m --random-route -b python_buildpack`を`manifest.yml`で表すと、

``` yaml
---
applications:
  - name: hello
    memory: 32m
    buildpack: python_buildpack
    random-route: true
```

となります。

このManifestファイルがあれば実行コマンドは`cf push`だけで良いです。

``` console
$ cf push
Using manifest file /Users/makit/cfws/python/manifest.yml
(以下、略)
```

