## PCF Devを用いたローカルCloud Foundry環境

**v0.16.0からPCF Devのインストール方法が変わりました。本ドキュメントをアップデートするまで、[公式ドキュメント](https://docs.pivotal.io/pcf-dev/)を参照してください。**

[PCF Dev](https://docs.pivotal.io/pcf-dev/index.html)を使うとVagrantを使って開発用にローカルで簡単にCloud Foundry環境を構築できます。

Pivotal Cloud Foundryが提供しているサービス(MySQL、Redis、RabbitMQ)も初めから組み込まれているため、ローカルでCloud Foundryを試したい場合に最適です。
(Pivotal Cloud FoundryやOSSのCloud Foundryとの違いは[こちら](https://docs.pivotal.io/pcf-dev/index.html)の「Comparing PCF Dev to Pivotal CF」を参照してください)

### PCF Devを使うために必要な環境

* [Vagrant](https://www.vagrantup.com/) 1.8以上
* [VirtualBox](https://www.virtualbox.org/) 5.0以上
* メモリの空き容量が4GB以上あるPC(8GB以上のPCを使用すること推奨します)

### セットアップ方法

PCF Devは[Pivotal Network](https://network.pivotal.io/products/pcfdev)からダウンロードできます。執筆時点での最新版は[v0.13.0](https://network.pivotal.io/products/pcfdev#/releases/1620)(Open Beta版)です。

<img width="1102" alt="スクリーンショット 0028-04-05 7.33.45.png" src="https://qiita-image-store.s3.amazonaws.com/0/1852/d520bb99-6266-47ec-9cef-1f6671650811.png">

ダウンロードするにはPivotal Networkにログインする必要があります。[こちら](https://network.pivotal.io/registrations/new)からアカウントを作成してください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/9a96d83c-8b7a-b0ce-b830-f9bfa03d141e.png)

ダウンロードした`pcfdev-v0.13.0.zip`を展開してください。

``` bash
$ unzip pcfdev-v0.13.0.zip
$ cd pcfdev-v0.13.0
$ ls -l
total 64
-rw-r--r--@ 1 makit  720748206  4466  3 30 12:26 Vagrantfile
-rwxr-xr-x@ 1 makit  720748206   129  4  1 00:52 destroy-osx
-rw-r--r--@ 1 makit  720748206   122  4  1 00:52 destroy-windows.ps1
-rwxr-xr-x  1 makit  720748206  2237  4  2 02:00 start-osx
-rw-r--r--@ 1 makit  720748206  2651  4  1 01:26 start-windows.ps1
-rwxr-xr-x@ 1 makit  720748206   123  4  1 00:52 stop-osx
-rw-r--r--@ 1 makit  720748206   116  4  1 00:52 stop-windows.ps1
```

`Vagrantfile`と起動、停止、破棄のためのスクリプトがOS X用、Windows用に用意されています。

スクリプトを使って`./start-osx`で起動できますが、ここではスクリプトを使わない方法を紹介します。スクリプトを使う方法は[マニュアル](https://docs.pivotal.io/pcf-dev/install-osx.html)を参照してください。

設定可能な環境変数は以下の通りです。

* `PCFDEV_IP` ... PCF DevのIPアドレス(デフォルトは`192.168.11.11`)
* `PCFDEV_DOMAIN` ... PCF Devのドメイン名(デフォルトのIPアドレスを使う場合は`local.pcfdev.io`、それ以外の場合は`$PCFDEV_IP.xip.io`)
* `VM_CORES` ... VMに割り当てるCPU数(デフォルトはホストマシンの論理コア数)
* `VM_MEMORY` ... VMに割り当てるメモリ(MB)(デフォルトはホストマシンの1/4のメモリ)

`192.168.11.*`をすでに使っている場合は、`PCFDEV_IP`を設定しないと、

``` console
The specified host network collides with a non-hostonly network!
This will cause your specified IP to be inaccessible. Please change
the IP or name of your host only network so that it no longer matches that of
a bridged or non-hostonly network.
```

と言われます。この場合は、以下の環境変数を設定してください。

``` console
$ export PCFDEV_IP=192.168.33.10
```

この設定を行った場合は、この後の`local.pcfdev.io`を`$PCFDEV_IP.xip.io`に読み替えてください。

以上の設定の後、`vagrant up`します。

``` console
$ vagrant up --provider virtualbox
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'pcfdev/pcfdev'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'pcfdev/pcfdev' is up to date...
==> default: Setting the name of the VM: pcfdev-v0130_default_1459809996734_18657
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: hostonly
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: 
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default: 
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
==> default: Configuring and enabling network interfaces...
==> default: Running provisioner: shell...
    default: Running: inline script
==> default: stdin: is not a tty
==> default: Waiting for services to start...
==> default: 0 out of 48 running
==> default: 0 out of 48 running
==> default: 0 out of 48 running
==> default: 0 out of 48 running
==> default: 0 out of 48 running
==> default: 4 out of 48 running
==> default: 28 out of 48 running
==> default: 45 out of 48 running
==> default: 45 out of 48 running
==> default: 46 out of 48 running
==> default: 46 out of 48 running
==> default: 48 out of 48 running
==> default: PCF Dev is now running.
==> default: To begin using PCF Dev, please run:
==> default: 	cf login -a api.local.pcfdev.io --skip-ssl-validation
==> default: Email: admin
==> default: Password: admin
```


## アプリケーションをデプロイ

まずはログインします。ユーザー名、パスワードともに`admin`です。

``` bash
$ cf login -a api.local.pcfdev.io -u admin -p admin --skip-ssl-validation
API endpoint: api.local.pcfdev.io
Authenticating...
OK

Targeted org pcfdev-org

Targeted space pcfdev-space


                   
API endpoint:   https://api.local.pcfdev.io (API version: 2.51.0)   
User:           admin   
Org:            pcfdev-org   
Space:          pcfdev-space 
```

`cf login`してしまえば、あとはこれまでのワークショップのコンテンツをそのまま試せます。
「[簡単なアプリケーションをデプロイ](deploy-application.md)」の内容を試してみましょう。


``` console
$ cf push
Using manifest file /Users/makit/git/hello-cf/manifest.yml

Creating app hello-tmaki in org pcfdev-org / space pcfdev-space as admin...
OK

Creating route hello-tmaki.local.pcfdev.io...
OK

Binding hello-tmaki.local.pcfdev.io to hello-tmaki...
OK

Uploading hello-tmaki...
Uploading app files from: /Users/makit/git/hello-cf/target/hello-cf-0.0.1-SNAPSHOT.jar
Uploading 492.8K, 89 files
Done uploading               
OK

Starting app hello-tmaki in org pcfdev-org / space pcfdev-space as admin...
Creating container
Successfully created container
Downloading app package...
Downloaded app package (11.8M)
Staging...
-----> Java Buildpack Version: v3.5.1 | https://github.com/cloudfoundry/java-buildpack#3abc3db
-----> Downloading Open Jdk JRE 1.8.0_65 from https://download.run.pivotal.io/openjdk/trusty/x86_64/openjdk-1.8.0_65.tar.gz (1m 25s)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (0.9s)
-----> Downloading Open JDK Like Memory Calculator 2.0.1_RELEASE from https://download.run.pivotal.io/memory-calculator/trusty/x86_64/memory-calculator-2.0.1_RELEASE.tar.gz (1.7s)
       Memory Settings: -XX:MaxMetaspaceSize=64M -XX:MetaspaceSize=64M -Xms382293K -Xss995K -Xmx382293K
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://download.run.pivotal.io/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (3.2s)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (44.7M)
Uploaded droplet (56.7M)
Uploading complete

0 of 3 instances running, 3 starting
0 of 3 instances running, 3 starting
0 of 3 instances running, 3 starting
0 of 3 instances running, 3 starting
3 of 3 instances running

App started


OK

App hello-tmaki was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.1_RELEASE -memorySizes=metaspace:64m.. -memoryWeights=heap:75,metaspace:10,native:10,stack:5 -memoryInitials=heap:100%,metaspace:100% -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/.:$PWD/.java-buildpack/spring_auto_reconfiguration/spring_auto_reconfiguration-1.10.0_RELEASE.jar org.springframework.boot.loader.JarLauncher`

Showing health and status for app hello-tmaki in org pcfdev-org / space pcfdev-space as admin...
OK

requested state: started
instances: 3/3
usage: 512M x 3 instances
urls: hello-tmaki.local.pcfdev.io
last uploaded: Tue Apr 5 05:20:54 UTC 2016
stack: cflinuxfs2
buildpack: https://github.com/cloudfoundry/java-buildpack#v3.5.1

     state     since                    cpu    memory           disk           details   
#0   running   2016-04-05 02:23:19 PM   0.0%   327.9M of 512M   136.2M of 1G      
#1   running   2016-04-05 02:23:19 PM   0.0%   318.7M of 512M   136.2M of 1G      
#2   running   2016-04-05 02:23:18 PM   0.0%   313.8M of 512M   136.2M of 1G 
```

``` console
$ for i in `seq 1 10`;do curl http://hello-tmaki.local.pcfdev.io;echo;done
Hello World! V3 (0)
Hello World! V3 (2)
Hello World! V3 (1)
Hello World! V3 (0)
Hello World! V3 (2)
Hello World! V3 (1)
Hello World! V3 (0)
Hello World! V3 (2)
Hello World! V3 (1)
Hello World! V3 (0)
```

マーケットプレイスにはv0.13.0の段階で

* MySQL
* Redis
* RabbitMQ

が登録されています。これは[Pivotal Services Suite for Pivotal Cloud Foundry](https://network.pivotal.io/products/pcf-services)とほぼ同じものです。(ただし、MySQLのHA対応はありません)
管理コンソールはありません。

``` bash
$ cf marketplace
Getting services from marketplace in org pcfdev-org / space pcfdev-space as admin...
OK

service      plans        description   
p-mysql      512mb, 1gb   MySQL databases on demand   
p-rabbitmq   standard     RabbitMQ is a robust and scalable high-performance multi-protocol messaging broker.   
p-redis      shared-vm    Redis service to provide a key-value store   

TIP:  Use 'cf marketplace -s SERVICE' to view descriptions of individual plans of a given service.
```

「[バックエンドサービスの利用](backend-service.md)」の内容をPCF Devで試すには、以下の`rediscloud`の代わりに`p-redis`を使ってください。
サービスインスタンス名は同じにすればmanifestファイルをそのまま利用できます。

``` console
$ cf create-service p-redis shared-vm myredis
```
