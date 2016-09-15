## PCF Devを用いたローカルCloud Foundry環境

**[公式ドキュメント](https://docs.pivotal.io/pcf-dev/)も参照してください。**

Cloud FoundryはオープンソースなPaaSプラットフォームであり、ローカルにもPaaS環境を構築することができます。


[PCF Dev](https://docs.pivotal.io/pcf-dev/index.html)は開発用にローカル環境で簡単にCloud Foundryを試すためのVM環境です。[Pivotal Cloud Foundry](https://pivotal.io/jp/platform)が提供しているサービス(MySQL、Redis、RabbitMQ)も初めから組み込まれていて、Virtual Boxだけで簡単にローカル開発環境を用意できます。(Pivotal Cloud FoundryやOSSのCloud Foundryとの違いは[こちら](https://docs.pivotal.io/pcf-dev/index.html)を参照してください)
v.0.15まではVagrantが必要でしたが、v.0.16からはVirtual BoxのみでOKです。

試したのは[v0.17.0](https://network.pivotal.io/products/pcfdev#/releases/1946)です。

## セットアップ方法

PCF Devは[Pivotal Network](https://network.pivotal.io/products/pcfdev)からダウンロードできます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/98e1bcf4-995b-96e4-a60b-737ec594a65c.png)


ダウンロードするにはPivotal Networkにログインする必要があります。[こちら](https://network.pivotal.io/registrations/new)からアカウントを作成してください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/9a96d83c-8b7a-b0ce-b830-f9bfa03d141e.png)

ダウンロードした`pcfdev-v0.17.0+PCF1.7.0-osx.zip`を展開してください(以下はOS Xの例)。

``` bash
$ unzip pcfdev-v0.17.0+PCF1.7.0-osx.zip 
Archive:  pcfdev-v0.17.0+PCF1.7.0-osx.zip
  inflating: pcfdev-v0.17.0+PCF1.7.0-osx 
```

`pcfdev-v0.17.0+PCF1.7.0-osx `はPCF DevをインストールするためのCloud Foundryのプラグインです。

[CF CLI](https://github.com/cloudfoundry/cli/releases)を使ってプラグインをインストール(`cf install-plugin`)したのち、プラグインのコマンド(`cf dev start`)を使ってPCF Devをインストールします。

途中でAPIトークンの入力を求められます。トークンはPivotal Networkの[プロフィール画面](https://network.pivotal.io/users/dashboard/edit-profile)の一番下に表示されています。


``` console
$ cf install-plugin ./pcfdev-v0.17.0+PCF1.7.0-osx

**注意: プラグインは必ずしも信頼できない作成者によって書かれたバイナリーです。プラグインのインストールと使用は自らの責任で行ってください。**

プラグイン ./pcfdev-v0.17.0+PCF1.7.0-osx をインストールしますか? (y または n)> y

プラグイン ./pcfdev-v0.17.0+PCF1.7.0-osx をインストールしています...
OK
プラグイン pcfdev v0.0.0 は正常にインストールされました。
$ cf dev start
Please retrieve your Pivotal Network API from:
https://network.pivotal.io/users/dashboard/edit-profile

API token> (APIトークンの入力)
BETA SOFTWARE END USER LICENSE AGREEMENT
(略)
Last Updated: April 14th, 2014

Accept (yes/no):> yes
Downloading VM...
Progress: |====================>| 100%
VM downloaded
Allocating 4096 MB out of 16384 MB total system memory (5500 MB free).
Importing VM...
Starting VM...
Provisioning VM...
Waiting for services to start...
9 out of 50 running
50 out of 50 running
 _______  _______  _______    ______   _______  __   __
|       ||       ||       |  |      | |       ||  | |  |
|    _  ||       ||    ___|  |  _    ||    ___||  |_|  |
|   |_| ||       ||   |___   | | |   ||   |___ |       |
|    ___||      _||    ___|  | |_|   ||    ___||       |
|   |    |     |_ |   |      |       ||   |___  |     |
|___|    |_______||___|      |______| |_______|  |___|
is now running.
To begin using PCF Dev, please run:
	cf login -a https://api.local.pcfdev.io --skip-ssl-validation
Admin user => Email: admin / Password: admin
Regular user => Email: user / Password: pass
```

立ち上がりました！

## アプリケーションをデプロイ

まずはログインします。ユーザー名、パスワードともに`admin`です。

``` bash
$ cf login -a https://api.local.pcfdev.io --skip-ssl-validation
API エンドポイント: https://api.local.pcfdev.io

Email> admin

Password> 
認証中です...
OK

組織を選択します (または Enter キーを押してスキップします):
1. pcfdev-org
2. system

Org> 1
組織 pcfdev-org をターゲットにしました

スペース pcfdev-space をターゲットにしました


                         
API エンドポイント:   https://api.local.pcfdev.io (API バージョン: 2.54.0)   
ユーザー:             admin   
組織:                 pcfdev-org   
スペース:             pcfdev-space 
```

[前に書いた入門記事](https://blog.ik.am/entries/359)と同じく`hello-pws`をpushします。

``` bash
$ git clone https://github.com/making/hello-pws
$ cd hello-pws
$ mvn clean package
$ cf push hello-pws -p target/hello-pws.jar -m 256m -b java_buildpack
admin としてアプリ hello-pws を組織 pcfdev-org / スペース pcfdev-space 内に作成しています...
OK

経路 hello-pws.local.pcfdev.io を作成しています...
OK

hello-pws.local.pcfdev.io を hello-pws にバインドしています...
OK

hello-pws をアップロードしています...
次のパスからアプリ・ファイルをアップロードしています: /var/folders/15/fww24j3d7pg9sz196cxv_6xm4nvlh8/T/unzipped-app203143786
485.1K、89 個のファイルをアップロードしています
Done uploading               
OK


admin として組織 pcfdev-org / スペース pcfdev-space 内のアプリ hello-pws を開始しています...
Downloading java_buildpack...
Downloaded java_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (11.7M)
Staging...
-----> Java Buildpack Version: v3.6 (offline) | https://github.com/cloudfoundry/java-buildpack.git#5194155
-----> Downloading Open Jdk JRE 1.8.0_71 from https://download.run.pivotal.io/openjdk/trusty/x86_64/openjdk-1.8.0_71.tar.gz (found in cache)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (2.7s)
-----> Downloading Open JDK Like Memory Calculator 2.0.1_RELEASE from https://download.run.pivotal.io/memory-calculator/trusty/x86_64/memory-calculator-2.0.1_RELEASE.tar.gz (found in cache)
       Memory Settings: -Xmx160M -XX:MaxMetaspaceSize=64M -Xss853K -Xms160M -XX:MetaspaceSize=64M
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://download.run.pivotal.io/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (107B)
Uploaded droplet (56.7M)
Uploading complete

1 個の中の 0 個のインスタンスが実行中です, 1 個が開始中です
1 個の中の 0 個のインスタンスが実行中です, 1 個が開始中です
1 個の中の 1 個のインスタンスが実行中です

アプリが開始されました


OK

アプリ hello-pws はこのコマンド `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.1_RELEASE -memorySizes=metaspace:64m.. -memoryWeights=heap:75,metaspace:10,native:10,stack:5 -memoryInitials=heap:100%,metaspace:100% -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/.:$PWD/.java-buildpack/spring_auto_reconfiguration/spring_auto_reconfiguration-1.10.0_RELEASE.jar org.springframework.boot.loader.JarLauncher` を使用して開始されました

admin として組織 pcfdev-org / スペース pcfdev-space 内のアプリ hello-pws の正常性と状況を表示しています...
OK

要求された状態: started
インスタンス: 1/1
使用法: 256M x 1 インスタンス
URL: hello-pws.local.pcfdev.io
最後アップロード日時: Sun Jul 3 11:43:18 UTC 2016
スタック: unknown
ビルドパック: java_buildpack

     状態   次の日時から             CPU    メモリー           ディスク            詳細   
#0   実行   2016-07-03 08:44:01 PM   0.0%   256M の中の 848K   512M の中の 16.4M 
```

デプロイできました。

``` bash
$ curl hello-pws.local.pcfdev.io
Hello from 10.0.2.15:60012
```

スケールアウトも[前記事](https://blog.ik.am/entries/359)と同じようにできます。

ローカルでCloud Foundryを色々試したい場合に便利です。

ただし、管理コンソールはありません。

またマーケットプレイスにはv0.17.0の段階で

* MySQL
* Redis
* RabbitMQ

が登録されています。これは[Pivotal Services Suite for Pivotal Cloud Foundry](https://network.pivotal.io/products/pcf-services)とほぼ同じもので、PCF Devで動いたアプリがPCFでも動くことを目的としてサービスが用意されているようです。

``` bash
$ cf marketplace
admin として組織 pcfdev-org / スペース pcfdev-space 内のマーケットプレイスからサービスを取得しています...
OK

サービス     プラン       説明   
p-mysql      512mb, 1gb   MySQL databases on demand   
p-rabbitmq   standard     RabbitMQ is a robust and scalable high-performance multi-protocol messaging broker.   
p-redis      shared-vm    Redis service to provide a key-value store   

ヒント:  特定のサービスの個々のプランの説明を表示するには、'cf marketplace -s SERVICE' を使用します。
```

### 管理コンソール(Apps Manager)にアクセスする

PCF Dev 0.17からはPivotal Cloud Foundryの売りの一つである管理コンソール(Apps Manager)も付いてきます。

[https://console.local.pcfdev.io/2](https://console.local.pcfdev.io/2)にアクセスすれば、デプロイされているアプリケーションの状態を確認できます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/57e6ad54-2130-c777-2cd2-6dcd89b0821a.png)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/82f33ec8-8318-6eee-08a4-3faf4dfd0f59.png)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/80391ccc-f1c7-4f45-c9e7-c486f3bd3abe.png)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/f0dc76d1-00b7-5494-4ad5-caac15abed85.png)



ちなみにApps Manager自体、このPCF Devのランタイムにデプロイされています。`system` Organizationにデプロイされているので`cf target`でOrganizationを切り替えれるとApps Managerの状態を確認できます。

``` console
$ cf target -o system
                         
API エンドポイント:   https://api.local.pcfdev.io (API バージョン: 2.54.0)   
ユーザー:             admin   
組織:                 system   
スペース:             system   
$ cf a
admin として組織 system / スペース system 内のアプリを取得しています...
OK

名前           要求された状態   インスタンス   メモリー   ディスク   URL   
apps-manager   started          6/6            64M        512M       apps-manager.local.pcfdev.io, console.local.pcfdev.io
```

## PCF Dev (v0.18.0以下)でSpring Boot 1.4を使う場合

Spring Boot 1.4からはjarのレイアウトが変わり、Cloud Foundryで動かすには[Java Buildpack 3.7以上が必要](https://github.com/pivotal-cf/pcfdev/issues/130)になります。Cloud Foundry側で用意されているBuildpackのバージョンは`cf buildpacks`で確認できますが、PCD Dev v0.18.0以下では3.6が用意されています。

この場合、Spring Boot 1.4のアプリをpushする際に3.7以上のbuildpackを`-b`で指定してください。

``` console
$ cf push ... -b https://github.com/cloudfoundry/java-buildpack.git#v3.9
```

`manifest.yml`を使う場合は次のようにbuildpackを指定してください。

``` yaml
---
applications:
  - name: ...
    # ...
    buildpack: https://github.com/cloudfoundry/java-buildpack#v3.9
```
