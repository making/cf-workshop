## スケールアウト (Java編)

本章では[バックエンドサービスRedisの利用](backend-service-redis_java.md)で作成したプロジェクトを利用します。

Cloud Foundryではスケールアウトも簡単です。`cf scale -i <Instance Count> <App>`で指定したインスタンス数にスケールアウトできます。


``` console
$ cf scale -i 2 hello-redis-tmaki
```

ログを見て、2つ目のインスタンスであることを示す`[APP/1]`が出力されていることを確認してください。

``` console
$ cf logs hello-redis-tmaki --recent
...
2016-03-23T17:15:23.29+0900 [API/5]      OUT Updated app with guid 77e2f12a-8c1c-4efb-9258-0f8020b4ca27 ({"instances"=>2})
2016-03-23T17:15:23.54+0900 [CELL/1]     OUT Creating container
2016-03-23T17:15:31.15+0900 [APP/1]      OUT   .   ____          _            __ _ _
2016-03-23T17:15:31.15+0900 [APP/1]      OUT  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
2016-03-23T17:15:31.15+0900 [APP/1]      OUT  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
2016-03-23T17:15:31.15+0900 [APP/1]      OUT   '  |____| .__|_| |_|_| |_\__, | / / / /
2016-03-23T17:15:31.15+0900 [APP/1]      OUT  =========|_|==============|___/=/_/_/_/
2016-03-23T17:15:31.15+0900 [APP/1]      OUT  :: Spring Boot ::        (v1.3.3.RELEASE)
...
```

`cf apps`で対象のインスタンス数が`2/2`になっていることを確認できます。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name                requested state   instances   memory   disk   urls   
hello-redis-tmaki   started           2/2         1G       1G     hello-redis-tmaki.cfapps.io
```

PWSの試用期間中はインスタンスに割り当てるメモリの合計が2GB以下にしないといけない制限があるので、最初にデプロイした`hello-tmaki`を削除していない場合は、エラーになります。

1インスタンスあたりのメモリは`cf scale -m <Memory <App>`で変更できます。

``` console
$ cf scale -m 512m hello-redis-tmaki
```

> Java Buildpack 4以上でメモリを512MBにしたい場合は次の環境変数を設定してください。（注意： Production環境では使わないでください）
>
> cf set-env hello-redis JAVA_OPTS '-XX:ReservedCodeCacheSize=32M -XX:MaxDirectMemorySize=32M'
> cf set-env hello-redis JBP_CONFIG_OPEN_JDK_JRE '{ memory_calculator: { stack_threads: 30 } }'
> cf scale -m 512m hello-redis

メモリを変更する場合はアプリケーションの再起動が必要な点に注意してください。

インスタンス数やメモリ数はPWSの管理画面からも変更できます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/fae8e9f5-10d9-9533-bcd7-620f6e912546.png)

Statusの表にインスタンスの情報が反映されます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/f705c419-76fc-6231-9f7f-355f951220c2.png)

Cloud Foundry内のRouterによってHTTPリクエストは2インスタンスに分散されますが、
インスタンス間でRedisキャッシュが共有されているため、出力結果は依然として同じです。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/5f9e014c-e422-6882-ba82-3a66f4c4462b.png)

折角なので、どちらのインスタンスにアクセスしているか分かるようにソースコードを修正しましょう。

``` java
    @GetMapping("/")
    String hello() {
        return greeter.hello() + " (" + System.getenv("CF_INSTANCE_INDEX") + ")"; // この行を変更
    }
```

今度はメモリ512MB、インスタンス数2を指定して`cf push`します。

``` console
$ ./mvnw package -Dmaven.test.skip=true
$ cf push hello-redis-tmaki -p target/hello-redis-0.0.1-SNAPSHOT.jar -m 512m -i 2
```

異なるインスタンスにアクセスしていることがわかります。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/6db798b2-9534-1d80-4a1b-d22ef7f02f58.png)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/7a848882-f164-d0b0-6f0e-1f242b848499.png)

4インスタンスにスケールアウトしましょう。

``` console
$ cf scale -i 4 hello-redis-tmaki
```

しばらくすると3番目と4番目のインスタンスにもアクセスできていることがわかります。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/ba4cd5ca-157c-7feb-a58f-c0b1117eae85.png)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/448d0cd6-bbf0-cef9-b52a-b4dc11b5c982.png)

以下の環境変数を設定して、アプリケーションのシャットダウンを有効にしましょう。

``` console
$ cf set-env hello-redis-tmaki management.security.enabled false
$ cf set-env hello-redis-tmaki endpoints.shutdown.enabled true
$ cf restart hello-redis-tmaki
```

インスタンス0~3までアクセスできることを確認した後、
以下のコマンドを3回実行してください。

```
$ curl -X POST http://hello-redis-tmaki.cfapps.io/shutdown
{"message":"Shutting down, bye..."}
```

4つのインスタンスのうち3つのインスタンスがダウンしたため、アプリケーションにアクセスすると残り1つのインスタンスからのみレスポンスがあります。

Cloud Foundryは指定したインスタンス数を保つために自動でインスタンスを再起動させます。しばらくすると再び4つインスタンスからレスポンスがあることを確認できるでしょう。
