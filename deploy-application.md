## 簡単なアプリケーションをデプロイ

Spring Bootを使ってとても簡単なWebアプリケーションを作成 & デプロイしましょう。

### プロジェクトの作成

以下のコマンドを実行すると、`hello-cf`フォルダに雛形プロジェクトが生成されます。

``` console
$ curl start.spring.io/starter.tgz \
       -d artifactId=hello-cf \
       -d baseDir=hello-cf \
       -d dependencies=web,actuator \
       -d applicationName=HelloCfApplication | tar -xzvf -
```

生成されたプロジェクトのソースを少しだけ修正します。任意のエディタ、IDEで
`hello-cf/src/main/java/com/example/HelloCfApplication.java`を開いてください。

``` java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
// ここから追加
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
// ここまで

@SpringBootApplication
@RestController // 追加
public class HelloCfApplication {

        // ここから追加
        @RequestMapping("/") 
        String hello() {
                return "Hello World!";
        }
        // ここまで

        public static void main(String[] args) {
                SpringApplication.run(HelloCfApplication.class, args);
        }
}
```

ソースコードを修正したら、アプリケーションをビルドします。

``` console
$ cd hello-cf
$ ./mvnw clean package
```

まずはローカルでアプリケーションを実行してみましょう。

``` console
$ java -jar target/hello-cf-0.0.1-SNAPSHOT.jar
```

[http://localhost:8080](http://localhost:8080)にアクセスしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/02020313-0016-ba4e-e2bd-f4ddd5db40f0.png)

Hello World!が表示されれば成功です。

### アプリケーションをCloud FoundryにPush

ビルドしたアプリケーションをCloud FoundryにPushしましょう。
`cf`コマンドを使えばとても簡単です。以下のコマンドを実行してください。

``` console
$ cf push hello-<your name> -p target/hello-cf-0.0.1-SNAPSHOT.jar
```

`<your name>`は自分の名前などを置換して、一意にしてください。以下では`<your name>`を`tmaki`とします。適宜自分の名前に読み替えてください。


``` console
$ cf push hello-tmaki -p target/hello-cf-0.0.1-SNAPSHOT.jar
Creating app hello-tmaki in org tmaki / space development as ****@gmail.com...
OK

Creating route hello-tmaki.cfapps.io...
OK

Binding hello-tmaki.cfapps.io to hello-tmaki...
OK

Uploading hello-tmaki...
Uploading app files from: target/hello-cf-0.0.1-SNAPSHOT.jar
Uploading 490.1K, 88 files
Done uploading               
OK

Starting app hello-tmaki in org tmaki / space development as ****@gmail.com...
Downloading nodejs_buildpack...
Downloading staticfile_buildpack...
Downloading ruby_buildpack...
Downloading java_buildpack...
Downloading go_buildpack...
Downloading python_buildpack...
Downloading php_buildpack...
Downloading liberty_buildpack...
Downloading binary_buildpack...
Downloaded staticfile_buildpack
Downloaded go_buildpack
Downloaded binary_buildpack
Downloaded nodejs_buildpack
Downloaded java_buildpack
Downloaded ruby_buildpack
Downloaded liberty_buildpack
Downloaded php_buildpack
Downloaded python_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (11.4M)
Staging...
-----> Java Buildpack Version: v3.6 | https://github.com/cloudfoundry/java-buildpack.git#5194155
-----> Downloading Open Jdk JRE 1.8.0_73 from https://download.run.pivotal.io/openjdk/trusty/x86_64/openjdk-1.8.0_73.tar.gz (1.4s)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.0s)
-----> Downloading Open JDK Like Memory Calculator 2.0.1_RELEASE from https://download.run.pivotal.io/memory-calculator/trusty/x86_64/memory-calculator-2.0.1_RELEASE.tar.gz (0.1s)
       Memory Settings: -Xms768M -Xss1M -Xmx768M -XX:MaxMetaspaceSize=104857K -XX:MetaspaceSize=104857K
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://download.run.pivotal.io/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (0.2s)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (44.7M)
Uploaded droplet (56.4M)
Uploading complete

0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App hello-tmaki was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.1_RELEASE -memorySizes=metaspace:64m.. -memoryWeights=heap:75,metaspace:10,native:10,stack:5 -memoryInitials=heap:100%,metaspace:100% -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/.:$PWD/.java-buildpack/spring_auto_reconfiguration/spring_auto_reconfiguration-1.10.0_RELEASE.jar org.springframework.boot.loader.JarLauncher`

Showing health and status for app hello-tmaki in org tmaki / space development as ****@gmail.com...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: hello-tmaki.cfapps.io
last uploaded: Thu Mar 17 07:34:03 UTC 2016
stack: cflinuxfs2
buildpack: java-buildpack=v3.6-https://github.com/cloudfoundry/java-buildpack.git#5194155 java-main open-jdk-like-jre=1.8.0_73 open-jdk-like-memory-calculator=2.0.1_RELEASE spring-auto-reconfiguration=1.10.0_RELEASE

     state     since                    cpu     memory         disk           details   
#0   running   2016-03-17 04:35:50 PM   10.3%   117.9M of 1G   135.8M of 1G
````

これでデプロイに成功しました。
`cf apps`でデプロイされているアプリケーションの一覧を取得できます。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name          requested state   instances   memory   disk   urls   
hello-tmaki   started           1/1         1G       1G     hello-tmaki.cfapps.io <---
```

`urls`の列に出力されている値がアプリケーションのURLです。この場合は[http://hello-tmaki.cfapps.io](http://hello-tmaki.cfapps.io)です。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/79860de1-4846-c922-8583-787e77a185d2.png)

Cloud Foundry上にデプロイされたアプリケーションにもアクセスできました。

[http://hello-tmaki.cfapps.io/env](http://hello-tmaki.cfapps.io/env)にアクセスすると環境変数やプロパティを確認できます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/dfafa4f4-0ec1-e568-ba5e-3d5cfaa799a5.png)

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
Connected, dumping recent logs for app hello-tmaki in org tmaki / space development as ****@gmail.com...

2016-03-23T14:35:03.24+0900 [STG/0]      OUT Downloading ruby_buildpack...
...
2016-03-23T14:35:03.60+0900 [STG/0]      OUT Creating container
2016-03-23T14:35:04.27+0900 [STG/0]      OUT Successfully created container
2016-03-23T14:35:04.27+0900 [STG/0]      OUT Downloading app package...
2016-03-23T14:35:05.14+0900 [STG/0]      OUT Downloaded app package (11.8M)
2016-03-23T14:35:05.14+0900 [STG/0]      OUT Downloading build artifacts cache...
2016-03-23T14:35:06.87+0900 [STG/0]      OUT Downloaded build artifacts cache (44.7M)
2016-03-23T14:35:06.87+0900 [STG/0]      OUT Staging...
2016-03-23T14:35:07.52+0900 [APP/0]      OUT 2016-03-23 05:35:07.500  INFO 14 --- [       Thread-3] ationConfigEmbeddedWebApplicationContext : Closing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@5cdaddaf: startup date [Fri Mar 18 19:57:58 UTC 2016]; root of context hierarchy
2016-03-23T14:35:07.59+0900 [APP/0]      OUT 2016-03-23 05:35:07.590  INFO 14 --- [       Thread-3] o.s.c.support.DefaultLifecycleProcessor  : Stopping beans in phase 0
2016-03-23T14:35:07.68+0900 [APP/0]      OUT 2016-03-23 05:35:07.685  INFO 14 --- [       Thread-3] o.s.j.e.a.AnnotationMBeanExporter        : Unregistering JMX-exposed beans on shutdown
2016-03-23T14:35:07.94+0900 [APP/0]      OUT Exit status 143
2016-03-23T14:35:08.46+0900 [STG/0]      OUT -----> Java Buildpack Version: v3.6 | https://github.com/cloudfoundry/java-buildpack.git#5194155
2016-03-23T14:35:08.64+0900 [STG/0]      OUT -----> Downloading Open Jdk JRE 1.8.0_73 from https://download.run.pivotal.io/openjdk/trusty/x86_64/openjdk-1.8.0_73.tar.gz (found in cache)
2016-03-23T14:35:09.70+0900 [STG/0]      OUT        Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.0s)
2016-03-23T14:35:09.72+0900 [STG/0]      OUT -----> Downloading Open JDK Like Memory Calculator 2.0.1_RELEASE from https://download.run.pivotal.io/memory-calculator/trusty/x86_64/memory-calculator-2.0.1_RELEASE.tar.gz (found in cache)
2016-03-23T14:35:09.75+0900 [STG/0]      OUT        Memory Settings: -Xms768M -XX:MetaspaceSize=104857K -Xss1M -Xmx768M -XX:MaxMetaspaceSize=104857K
2016-03-23T14:35:09.76+0900 [STG/0]      OUT -----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://download.run.pivotal.io/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
2016-03-23T14:35:18.57+0900 [STG/0]      OUT Exit status 0
2016-03-23T14:35:18.57+0900 [STG/0]      OUT Staging complete
2016-03-23T14:35:18.57+0900 [STG/0]      OUT Uploading droplet, build artifacts cache...
2016-03-23T14:35:18.57+0900 [STG/0]      OUT Uploading droplet...
2016-03-23T14:35:18.57+0900 [STG/0]      OUT Uploading build artifacts cache...
2016-03-23T14:35:19.79+0900 [STG/0]      OUT Uploaded build artifacts cache (44.7M)
2016-03-23T14:35:41.65+0900 [STG/0]      OUT Uploaded droplet (56.7M)
2016-03-23T14:35:41.66+0900 [STG/0]      OUT Uploading complete
2016-03-23T14:35:42.65+0900 [CELL/0]     OUT Creating container
2016-03-23T14:35:43.22+0900 [CELL/0]     OUT Successfully created container
2016-03-23T14:35:46.20+0900 [CELL/0]     OUT Starting health monitoring of container
2016-03-23T14:35:47.88+0900 [APP/0]      OUT   .   ____          _            __ _ _
2016-03-23T14:35:47.88+0900 [APP/0]      OUT  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
2016-03-23T14:35:47.88+0900 [APP/0]      OUT ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
2016-03-23T14:35:47.88+0900 [APP/0]      OUT  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
2016-03-23T14:35:47.88+0900 [APP/0]      OUT   '  |____| .__|_| |_|_| |_\__, | / / / /
2016-03-23T14:35:47.88+0900 [APP/0]      OUT  =========|_|==============|___/=/_/_/_/
2016-03-23T14:35:47.89+0900 [APP/0]      OUT  :: Spring Boot ::        (v1.3.3.RELEASE)
2016-03-23T14:35:48.08+0900 [APP/0]      OUT 2016-03-23 05:35:48.081  INFO 17 --- [           main] pertySourceApplicationContextInitializer : Adding 'cloud' PropertySource to ApplicationContext
2016-03-23T14:35:48.13+0900 [APP/0]      OUT 2016-03-23 05:35:48.138  INFO 17 --- [           main] nfigurationApplicationContextInitializer : Adding cloud service auto-reconfiguration to ApplicationContext
2016-03-23T14:35:48.15+0900 [APP/0]      OUT 2016-03-23 05:35:48.152  INFO 17 --- [           main] com.example.HelloCfApplication           : Starting HelloCfApplication on f8e414fd07r with PID 17 (/home/vcap/app started by vcap in /home/vcap/app)
2016-03-23T14:35:48.15+0900 [APP/0]      OUT 2016-03-23 05:35:48.152  INFO 17 --- [           main] com.example.HelloCfApplication           : The following profiles are active: cloud
2016-03-23T14:35:48.21+0900 [APP/0]      OUT 2016-03-23 05:35:48.212  INFO 17 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@e17c606: startup date [Wed Mar 23 05:35:48 UTC 2016]; root of context hierarchy
...
2016-03-23T14:35:52.78+0900 [APP/0]      OUT 2016-03-23 05:35:52.783  INFO 17 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2016-03-23T14:35:52.79+0900 [APP/0]      OUT 2016-03-23 05:35:52.790  INFO 17 --- [           main] com.example.HelloCfApplication           : Started HelloCfApplication in 5.935 seconds (JVM running for 6.547)
2016-03-23T14:35:53.24+0900 [CELL/0]     OUT Container became healthy

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
$ cf push hello -p target/hello-cf-0.0.1-SNAPSHOT.jar --random-route
```

`cf apps`を確認すると、ホスト名が`hello-mooned-falsification`になっていることがわかります。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name    requested state   instances   memory   disk   urls   
hello   started           1/1         1G       1G     hello-mooned-falsification.cfapps.io 
```

この場合、[http://hello-mooned-falsification.cfapps.io](
http://hello-mooned-falsification.cfapps.io)にアクセスできます。


### Buildpackを指定する

`cf push`でアプリケーションをアップロードした後、ステージングとよばれるフェーズでランタイム(JREやサーバーなど)を追加し実行可能なDropletという形式を作成します。Dropletを作るためのBuildpackとよばれる仕組みが使われます。

アップロードしたファイル群(アーティファクト)から自動で適用すべきBuildpackが判断され、Cloud Foundryに他言語対応はここで行われています。

利用可能なBuildpack一覧は`cf buildpacks`で取得できます。

``` console
$ cf buildpacks
Getting buildpacks...

buildpack              position   enabled   locked   filename   
staticfile_buildpack   1          true      false    staticfile_buildpack-cached-v1.3.1.zip   
java_buildpack         2          true      false    java-buildpack-v3.6.zip   
ruby_buildpack         3          true      false    ruby_buildpack-cached-v1.6.13.zip   
nodejs_buildpack       4          true      false    nodejs_buildpack-cached-v1.5.5.zip   
go_buildpack           5          true      false    go_buildpack-cached-v1.7.2.zip   
python_buildpack       6          true      false    python_buildpack-cached-v1.5.4.zip   
php_buildpack          7          true      false    php_buildpack-cached-v4.3.5.zip   
liberty_buildpack      8          true      false    liberty_buildpack.zip   
binary_buildpack       9          true      false    binary_buildpack-cached-v1.0.1.zip 
```

デフォルトでは、`cf push`でアーティファクトをアップロードした後、利用可能なBuildpackを全てダウンロードし、優先順(`position`順)にチェックし、対象のBuildpackを特定しDroplet(実行可能な形式)を作成します。

今回の場合は、jarファイルからjava_buildpackが検知され、かつjarの内部に`lib/spring-boot-.*.jar`が存在することからSpring Boot用のDropletが作成されます。

Buildpackは`-b`で明示的に指定できます。明示することで自動検出のための時間を短縮できます。

``` console
$ cf push hello -p target/hello-cf-0.0.1-SNAPSHOT.jar --random-route -b java_buildpack
Updating app hello in org tmaki / space development as ****@gmail.com...
OK

Uploading hello...
Uploading app files from: target/hello-cf-0.0.1-SNAPSHOT.jar
Uploading 490.1K, 88 files
Done uploading               
OK

Stopping app hello in org tmaki / space development as ****@gmail.com...
OK

Starting app hello in org tmaki / space development as ****@gmail.com...
Downloading java_buildpack...
Downloaded java_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (11.4M)
Downloading build artifacts cache...
Downloaded build artifacts cache (44.7M)
Staging...
-----> Java Buildpack Version: v3.6 | https://github.com/cloudfoundry/java-buildpack.git#5194155
-----> Downloading Open Jdk JRE 1.8.0_73 from https://download.run.pivotal.io/openjdk/trusty/x86_64/openjdk-1.8.0_73.tar.gz (found in cache)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.0s)
-----> Downloading Open JDK Like Memory Calculator 2.0.1_RELEASE from https://download.run.pivotal.io/memory-calculator/trusty/x86_64/memory-calculator-2.0.1_RELEASE.tar.gz (found in cache)
       Memory Settings: -Xmx768M -XX:MaxMetaspaceSize=104857K -Xss1M -Xms768M -XX:MetaspaceSize=104857K
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://download.run.pivotal.io/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (44.7M)
Uploaded droplet (56.4M)
Uploading complete

0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App hello was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.1_RELEASE -memorySizes=metaspace:64m.. -memoryWeights=heap:75,metaspace:10,native:10,stack:5 -memoryInitials=heap:100%,metaspace:100% -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/.:$PWD/.java-buildpack/spring_auto_reconfiguration/spring_auto_reconfiguration-1.10.0_RELEASE.jar org.springframework.boot.loader.JarLauncher`

Showing health and status for app hello in org tmaki / space development as ****@gmail.com...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: hello-mooned-falsification.cfapps.io
last uploaded: Thu Mar 17 08:23:17 UTC 2016
stack: cflinuxfs2
buildpack: java_buildpack

     state     since                    cpu    memory         disk           details   
#0   running   2016-03-17 05:24:00 PM   0.0%   421.8M of 1G   135.8M of 1G
```

### Manifestファイルを作成

ここまで`cf`コマンドで指定してきたオプションは`manifest.yml`というyamlファイルに定義できます。

`cf push hello -p target/hello-cf-0.0.1-SNAPSHOT.jar --random-route -b java_buildpack`を`manifest.yml`で表すと、

``` yaml
---
applications:
  - name: hello
    path: target/hello-cf-0.0.1-SNAPSHOT.jar
    buildpack: java_buildpack
    random-route: true
```

となります。

このManifestファイルがあれば実行コマンドは`cf push`だけで良いです。

``` console
$ cf push
Using manifest file /Users/makit/git/hello-cf/manifest.yml
(以下、略)
```

