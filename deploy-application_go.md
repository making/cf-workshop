## 簡単なアプリケーションをデプロイ (Go編)

[net/http](https://golang.org/pkg/net/http/)を使ってとても簡単なWebアプリケーションを作成 & デプロイしましょう。
パッケージマネージャーとして[Glide](https://glide.sh/)がインストールされていることを前提とします。

### プロジェクトの作成

`hello-cf`フォルダを作成してください。

``` console
$ mkdir -p $GOPATH/src/github.com/<your github account>/hello-cf
$ cd $GOPATH/src/github.com/<your github account>/hello-cf
```

``` bash
$ glide init
[INFO] Generating a YAML configuration file and guessing the dependencies
[INFO] Attempting to import from other package managers (use --skip-import to skip)
```

`main.go`を作成して、次の内容を記述してください。

``` go
package main

import (
	"fmt"
	"net/http"
	"os"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprint(w, "Hello World!")
	})
	http.ListenAndServe(":"+os.Getenv("PORT"), nil)
}
```

まずはローカルでアプリケーションを実行してみましょう。

``` console
$ PORT=8080 go run main.go
```

Windowsのコマンドプロンプトの場合は

``` console
$ set PORT=8080
$ go run main.go
```

[http://localhost:8080](http://localhost:8080)にアクセスしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/02020313-0016-ba4e-e2bd-f4ddd5db40f0.png)

Hello World!が表示されれば成功です。

### アプリケーションをCloud FoundryにPush

作成したアプリケーションをCloud FoundryにPushしましょう。

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
$ cf push hello-tmaki -m 8m
Creating app hello-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK

Creating route hello-tmaki.cfapps.io...
OK

Binding hello-tmaki.cfapps.io to hello-tmaki...
OK

Uploading hello-tmaki...
Uploading app files from: /Users/makit/go/src/github.com/making/hello-cf
Uploading 472B, 2 files
Done uploading               
OK

Starting app hello-tmaki in org APJ / space staging as tmaki@pivotal.io...
Downloading dotnet_core_buildpack_beta...
Downloading java_buildpack...
Downloading python_buildpack...
Downloaded ruby_buildpack
Downloaded dotnet_core_buildpack_beta
Downloading nodejs_buildpack...
Downloaded staticfile_buildpack
Downloading liberty_buildpack...
Downloading go_buildpack...
Downloaded java_buildpack
Downloading binary_buildpack...
Downloaded nodejs_buildpack
Downloading php_buildpack...
Downloaded python_buildpack
Downloaded php_buildpack
Downloaded binary_buildpack
Downloaded go_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (530B)
Staging...
-------> Buildpack version 1.7.12
file:///tmp/buildpacks/8d99fe2fe1e84e0e9a877cc40ea7166e/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_godep_godep-v74-linux-x64.tgz
file:///tmp/buildpacks/8d99fe2fe1e84e0e9a877cc40ea7166e/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_glide_glide-v0.11.1-linux-x64.tgz
-----> Installing go1.6.3... done
Downloaded [file:///tmp/buildpacks/8d99fe2fe1e84e0e9a877cc40ea7166e/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_go_go1.6.3.linux-amd64.tar.gz]
 !!    Installing package '.' (default)
-----> Fetching any unsaved dependencies (glide install)
[INFO]	Lock file (glide.lock) does not exist. Performing update.
[INFO]	Downloading dependencies. Please wait...
[INFO]	No references set.
[INFO]	Resolving imports
[INFO]	Downloading dependencies. Please wait...
[INFO]	Setting references for remaining imports
[INFO]	No references set.
[INFO]	Project relies on 0 dependencies.
-----> Running: go install -v -tags cloudfoundry . 
github.com/making/hello-cf
Exit status 0
Uploading droplet, build artifacts cache...
Staging complete
Uploading build artifacts cache...
Uploading droplet...
Uploaded droplet (2.2M)
Uploaded build artifacts cache (65.6M)
Uploading complete
Destroying container
Successfully destroyed container

1 of 1 instances running

App started


OK

App hello-tmaki was started using this command `hello-cf`

Showing health and status for app hello-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK

requested state: started
instances: 1/1
usage: 8M x 1 instances
urls: hello-tmaki.cfapps.io
last uploaded: Sun Oct 2 07:23:47 UTC 2016
stack: cflinuxfs2
buildpack: Go

     state     since                    cpu    memory       disk         details
#0   running   2016-10-02 04:24:33 PM   0.0%   3.1M of 8M   8.6M of 1G
````

これでデプロイに成功しました。
`cf apps`でデプロイされているアプリケーションの一覧を取得できます。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name          requested state   instances   memory   disk   urls   
hello-tmaki   started           1/1         8M       1G     hello-tmaki.cfapps.io <---
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


> **ノート** 使用できるGoのバージョン
> 
> 使用できるGoのバージョンは使用するBuildpackによります。
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
> go_buildpack                 5          true      true     go_buildpack-cached-v1.7.12.zip <-----
> python_buildpack             6          true      true     python_buildpack-cached-v1.5.9.zip
> php_buildpack                7          true      true     php_buildpack-cached-v4.3.19.zip
> liberty_buildpack            8          true      true     liberty_buildpack.zip
> binary_buildpack             9          true      true     binary_buildpack-cached-v1.0.3.zip
> dotnet_core_buildpack_beta   10         true      true     dotnet-core_buildpack-cached-v1.0.0.zip
> ```
> 
> 上の例ではGoのBuildpackバージョンは1.7.12です。BuildpackのリリースページにそのBuildpackバージョンで利用出来るGoのバージョンが列挙されています。
> 
> https://github.com/cloudfoundry/go-buildpack/releases/tag/v1.7.12
> 
> Go Buildpack 1.7.12の場合はデフォルトのGoのバージョンは1.6.3です。Goのバージョンを変更したい場合は次のように環境変数`GOVERSION`に指定し、再度ステージングから行います。
>
> ``` console
> cf set-env hello-tmaki GOVERSION go1.7
> cf restage hello-tmaki
> ```


直近のログは`cf logs <App> --recent`で確認できます。

``` console
$ cf logs hello-tmaki --recent
Connected, dumping recent logs for app hello-tmaki in org APJ / space staging as tmaki@pivotal.io...

2016-10-02T16:23:38.58+0900 [API/3]      OUT Created app with guid 4b02def1-a607-4428-a873-22a9950b4135
2016-10-02T16:23:42.54+0900 [API/3]      OUT Updated app with guid 4b02def1-a607-4428-a873-22a9950b4135 ({"route"=>"b64c0430-4399-4939-b641-4e36a0ae2956", :verb=>"add", :relation=>:routes, :related_guid=>"b64c0430-4399-4939-b641-4e36a0ae2956"})
2016-10-02T16:23:59.73+0900 [API/5]      OUT Updated app with guid 4b02def1-a607-4428-a873-22a9950b4135 ({"state"=>"STARTED"})
2016-10-02T16:24:00.11+0900 [STG/0]      OUT Downloading dotnet_core_buildpack_beta...
2016-10-02T16:24:00.11+0900 [STG/0]      OUT Downloading java_buildpack...
2016-10-02T16:24:00.11+0900 [STG/0]      OUT Downloading python_buildpack...
2016-10-02T16:24:00.16+0900 [STG/0]      OUT Downloaded ruby_buildpack
2016-10-02T16:24:00.16+0900 [STG/0]      OUT Downloaded dotnet_core_buildpack_beta
2016-10-02T16:24:00.16+0900 [STG/0]      OUT Downloading nodejs_buildpack...
2016-10-02T16:24:00.16+0900 [STG/0]      OUT Downloaded staticfile_buildpack
2016-10-02T16:24:00.16+0900 [STG/0]      OUT Downloading liberty_buildpack...
2016-10-02T16:24:00.16+0900 [STG/0]      OUT Downloaded python_buildpack
2016-10-02T16:24:00.16+0900 [STG/0]      OUT Downloading php_buildpack...
2016-10-02T16:24:00.17+0900 [STG/0]      OUT Downloading go_buildpack...
2016-10-02T16:24:00.17+0900 [STG/0]      OUT Downloaded java_buildpack
2016-10-02T16:24:00.17+0900 [STG/0]      OUT Downloading binary_buildpack...
2016-10-02T16:24:00.20+0900 [STG/0]      OUT Downloaded nodejs_buildpack
2016-10-02T16:24:00.21+0900 [STG/0]      OUT Downloaded go_buildpack
2016-10-02T16:24:00.25+0900 [STG/0]      OUT Downloaded php_buildpack
2016-10-02T16:24:00.28+0900 [STG/0]      OUT Downloaded binary_buildpack
2016-10-02T16:24:00.28+0900 [STG/0]      OUT Creating container
2016-10-02T16:24:06.04+0900 [STG/0]      OUT Successfully created container
2016-10-02T16:24:06.04+0900 [STG/0]      OUT Downloading app package...
2016-10-02T16:24:06.23+0900 [STG/0]      OUT Downloaded app package (530B)
2016-10-02T16:24:06.23+0900 [STG/0]      OUT Staging...
2016-10-02T16:24:06.88+0900 [STG/0]      OUT -------> Buildpack version 1.7.12
2016-10-02T16:24:07.05+0900 [STG/0]      OUT file:///tmp/buildpacks/8d99fe2fe1e84e0e9a877cc40ea7166e/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_godep_godep-v74-linux-x64.tgz
2016-10-02T16:24:07.27+0900 [STG/0]      OUT file:///tmp/buildpacks/8d99fe2fe1e84e0e9a877cc40ea7166e/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_glide_glide-v0.11.1-linux-x64.tgz
2016-10-02T16:24:09.90+0900 [STG/0]      OUT -----> Installing go1.6.3... done
2016-10-02T16:24:09.90+0900 [STG/0]      OUT Downloaded [file:///tmp/buildpacks/8d99fe2fe1e84e0e9a877cc40ea7166e/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_go_go1.6.3.linux-amd64.tar.gz]
2016-10-02T16:24:09.93+0900 [STG/0]      OUT  !!    Installing package '.' (default)
2016-10-02T16:24:09.93+0900 [STG/0]      OUT -----> Fetching any unsaved dependencies (glide install)
2016-10-02T16:24:09.96+0900 [STG/0]      OUT [INFO]	Lock file (glide.lock) does not exist. Performing update.
2016-10-02T16:24:09.96+0900 [STG/0]      OUT [INFO]	Downloading dependencies. Please wait...
2016-10-02T16:24:09.96+0900 [STG/0]      OUT [INFO]	No references set.
2016-10-02T16:24:09.96+0900 [STG/0]      OUT [INFO]	Resolving imports
2016-10-02T16:24:09.96+0900 [STG/0]      OUT [INFO]	Downloading dependencies. Please wait...
2016-10-02T16:24:09.96+0900 [STG/0]      OUT [INFO]	Setting references for remaining imports
2016-10-02T16:24:09.96+0900 [STG/0]      OUT [INFO]	No references set.
2016-10-02T16:24:09.96+0900 [STG/0]      OUT [INFO]	Project relies on 0 dependencies.
2016-10-02T16:24:09.97+0900 [STG/0]      OUT -----> Running: go install -v -tags cloudfoundry . 
2016-10-02T16:24:10.10+0900 [STG/0]      OUT github.com/making/hello-cf
2016-10-02T16:24:23.95+0900 [STG/0]      OUT Exit status 0
2016-10-02T16:24:23.95+0900 [STG/0]      OUT Staging complete
2016-10-02T16:24:23.95+0900 [STG/0]      OUT Uploading droplet, build artifacts cache...
2016-10-02T16:24:23.95+0900 [STG/0]      OUT Uploading build artifacts cache...
2016-10-02T16:24:23.95+0900 [STG/0]      OUT Uploading droplet...
2016-10-02T16:24:25.15+0900 [STG/0]      OUT Uploaded droplet (2.2M)
2016-10-02T16:24:25.16+0900 [STG/0]      OUT Uploaded build artifacts cache (65.6M)
2016-10-02T16:24:25.18+0900 [STG/0]      OUT Uploading complete
2016-10-02T16:24:25.27+0900 [STG/0]      OUT Destroying container
2016-10-02T16:24:25.92+0900 [CELL/0]     OUT Creating container
2016-10-02T16:24:27.38+0900 [STG/0]      OUT Successfully destroyed container
2016-10-02T16:24:30.48+0900 [CELL/0]     OUT Successfully created container
2016-10-02T16:24:30.94+0900 [CELL/0]     OUT Starting health monitoring of container
2016-10-02T16:24:33.02+0900 [CELL/0]     OUT Container became healthy
2016-10-02T16:26:19.72+0900 [RTR/1]      OUT hello-tmaki.cfapps.io - [02/10/2016:07:26:19.720 +0000] "GET / HTTP/1.1" 200 0 12 "http://qiita.com/drafts/b7fc193743bc87741bb2/edit" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36" 10.10.2.209:26284 x_forwarded_for:"202.241.169.198" x_forwarded_proto:"http" vcap_request_id:989f25b2-3e78-4804-4700-62c5cb83997e response_time:0.002343027 app_id:4b02def1-a607-4428-a873-22a9950b4135 app_index:0
2016-10-02T16:26:20.11+0900 [RTR/2]      OUT hello-tmaki.cfapps.io - [02/10/2016:07:26:20.104 +0000] "GET /favicon.ico HTTP/1.1" 200 0 12 "http://hello-tmaki.cfapps.io/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36" 10.10.2.209:19713 x_forwarded_for:"202.241.169.198" x_forwarded_proto:"http" vcap_request_id:33473bf7-c2fc-45c1-4f26-3ca590afb3ad response_time:0.005880919 app_id:4b02def1-a607-4428-a873-22a9950b4135 app_index:0
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
hello   started           1/1         8M       1G     hello-alphameric-laity.cfapps.io
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

今回の場合は、go_buildpackが検知され、Go用のDropletが作成されます。

Buildpackは`-b`で明示的に指定できます。明示することで自動検出のための時間を短縮できます。

``` console
$ cf push hello -m 8m --random-route -b go_buildpack
Creating app hello in org APJ / space staging as tmaki@pivotal.io...
OK

Creating route hello-noncataclysmic-glassiness.cfapps.io...
OK

Binding hello-noncataclysmic-glassiness.cfapps.io to hello...
OK

Uploading hello...
Uploading app files from: /Users/makit/go/src/github.com/making/hello-cf
Uploading 472B, 2 files
Done uploading               
OK

Starting app hello in org APJ / space staging as tmaki@pivotal.io...
Downloading go_buildpack...
Downloaded go_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (530B)
Staging...
-------> Buildpack version 1.7.12
file:///tmp/buildpacks/8d99fe2fe1e84e0e9a877cc40ea7166e/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_godep_godep-v74-linux-x64.tgz
file:///tmp/buildpacks/8d99fe2fe1e84e0e9a877cc40ea7166e/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_glide_glide-v0.11.1-linux-x64.tgz
-----> Installing go1.6.3... done
Downloaded [file:///tmp/buildpacks/8d99fe2fe1e84e0e9a877cc40ea7166e/dependencies/https___buildpacks.cloudfoundry.org_concourse-binaries_go_go1.6.3.linux-amd64.tar.gz]
 !!    Installing package '.' (default)
-----> Fetching any unsaved dependencies (glide install)
[INFO]	No references set.
[INFO]	Resolving imports
[INFO]	Downloading dependencies. Please wait...
[INFO]	Setting references for remaining imports
[INFO]	No references set.
[INFO]	Project relies on 0 dependencies.
[INFO]	Lock file (glide.lock) does not exist. Performing update.
[INFO]	Downloading dependencies. Please wait...
-----> Running: go install -v -tags cloudfoundry . 
github.com/making/hello-cf
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (65.6M)
Uploaded droplet (2.2M)
Uploading complete
Destroying container
Successfully destroyed container

1 of 1 instances running

App started


OK

App hello was started using this command `hello-cf`

Showing health and status for app hello in org APJ / space staging as tmaki@pivotal.io...
OK

requested state: started
instances: 1/1
usage: 8M x 1 instances
urls: hello-noncataclysmic-glassiness.cfapps.io
last uploaded: Sun Oct 2 07:38:40 UTC 2016
stack: cflinuxfs2
buildpack: go_buildpack

     state     since                    cpu    memory     disk         details
#0   running   2016-10-02 04:39:15 PM   0.0%   3M of 8M   8.6M of 1G
```

### Manifestファイルを作成

ここまで`cf`コマンドで指定してきたオプションは`manifest.yml`というyamlファイルに定義できます。

`cf push hello -m 8m --random-route -b go_buildpack`を`manifest.yml`で表すと、

``` yaml
---
applications:
  - name: hello
    memory: 8m
    buildpack: go_buildpack
    random-route: true
```

となります。

このManifestファイルがあれば実行コマンドは`cf push`だけで良いです。

``` console
$ cf push
Using manifest file /Users/makit/go/src/github.com/making/hello-cf/manifest.yml
(以下、略)
```

次のようにGoのバージョンを`manifest.yml`に指定することもできます。

``` yaml
---
applications:
  - name: hello
    memory: 8m
    buildpack: go_buildpack
    random-route: true
    env:
      GOVERSION: go1.7
```
