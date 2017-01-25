## スケールアウト (PHP編)

本章では[バックエンドサービスRedisの利用](backend-service-redis_php-simple.md)で作成したプロジェクトを利用します。

Cloud Foundryではスケールアウトも簡単です。`cf scale -i <Instance Count> <App>`で指定したインスタンス数にスケールアウトできます。


``` console
$ cf scale -i 2 hello-redis-tmaki
```

ログを見て、2つ目のインスタンスであることを示す`[APP/1]`が出力されていることを確認してください。

``` console
$ cf logs hello-redis-tmaki --recent
...
2017-01-25T20:40:43.38+0900 [API/0]      OUT Updated app with guid 4555ca52-2b7c-4fed-95bf-f41f91f54598 ({"instances"=>2})
2017-01-25T20:40:43.78+0900 [CELL/1]     OUT Creating container
2017-01-25T20:40:46.00+0900 [CELL/1]     OUT Successfully created container
2017-01-25T20:40:48.97+0900 [CELL/1]     OUT Starting health monitoring of container
2017-01-25T20:40:49.40+0900 [APP/PROC/WEB/1]OUT 11:40:49 php-fpm | [25-Jan-2017 11:40:49] NOTICE: fpm is running, pid 50
2017-01-25T20:40:49.40+0900 [APP/PROC/WEB/1]OUT 11:40:49 php-fpm | [25-Jan-2017 11:40:49] NOTICE: ready to handle connections
2017-01-25T20:40:49.47+0900 [APP/PROC/WEB/1]OUT 11:40:49 httpd   | [Wed Jan 25 11:40:49.455486 2017] [mpm_event:notice] [pid 48:tid 140279445337984] AH00489: Apache/2.4.25 (Unix) configured -- resuming normal operations
2017-01-25T20:40:49.47+0900 [APP/PROC/WEB/1]OUT 11:40:49 httpd   | [Wed Jan 25 11:40:49.455620 2017] [mpm_event:info] [pid 48:tid 140279445337984] AH00490: Server built: Dec 20 2016 22:27:46
2017-01-25T20:40:49.47+0900 [APP/PROC/WEB/1]OUT 11:40:49 httpd   | [Wed Jan 25 11:40:49.455636 2017] [core:notice] [pid 48:tid 140279445337984] AH00094: Command line: '/app/httpd/bin/httpd -f /home/vcap/app/httpd/conf/httpd.conf -D FOREGROUND'
2017-01-25T20:40:51.06+0900 [CELL/1]     OUT Container became healthy
...
```

`cf apps`で対象のインスタンス数が`2/2`になっていることを確認できます。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name                requested state   instances   memory   disk   urls   
hello-redis-tmaki   started           2/2         32M      1G     hello-redis-tmaki.cfapps.io
```

インスタンス数やメモリ数は[PWSの管理画面](https://console.run.pivotal.io)からも変更できます。Instancesの表にインスタンスの情報が反映されます。


![image](https://qiita-image-store.s3.amazonaws.com/0/1852/f0935b89-33c3-0d74-0145-b4983796805e.png)


Cloud Foundry内のRouterによってHTTPリクエストは2インスタンスに分散されますが、
インスタンス間でRedisキャッシュが共有されているため、出力結果は依然として同じです。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/3f860de6-685d-437d-9291-184261df4fd2.png)

折角なので、どちらのインスタンスにアクセスしているか分かるようにソースコードを修正しましょう。

``` php
echo 'Hello. It\'s ' . $now . ' now (' . getenv('CF_INSTANCE_INDEX') . ')';// この行を変更
```

再度`cf push`します。

``` console
$ cf push hello-redis-tmaki
```

異なるインスタンスにアクセスしていることがわかります。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/75a8108b-05e2-9cb0-355f-4d7e66f1228b.png)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/714709cf-230f-cdf2-76ab-649aa8b33f3c.png)

4インスタンスにスケールアウトしましょう。

``` console
$ cf scale -i 4 hello-redis-tmaki
```

しばらくすると3番目と4番目のインスタンスにもアクセスできていることがわかります。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/4f814b94-448d-6b5a-5df0-2152b0e2b6c6.png)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/628c7b32-f040-5946-08fa-05e945dbbdd1.png)

アプリケーションを次のように変更して、httpdをわざとシャットダウンできるようにしましょう。

``` php
<?php
// ここから追加
if (isset($_GET['kill'])) {
    exec('pkill -KILL httpd');
}
// ここまで

// Redisの接続情報を環境変数VCAP_SERVICESより取得
$vcap_services = json_decode(getenv('VCAP_SERVICES'));
$service_name = 'rediscloud';
$service = $vcap_services->$service_name;
$credentials = $service[0]->credentials;

// Redisに接続
$redis = new Redis();
$redis->connect($credentials->hostname, $credentials->port);
$redis->auth($credentials->password);

$now = $redis->get('time');
if (!$now) {
    $now = date('c');
    $redis->set('time', $now);
}
```

インスタンス0~3までアクセスできることを確認した後、[https://hello-redis-tmaki.cfapps.io/?kill](https://hello-redis-tmaki.cfapps.io/?kill)に3回アクセス、または
以下のコマンドを3回実行してください。

```
$ curl "https://hello-redis-tmaki.cfapps.io/?kill"
```

4つのインスタンスのうち3つのインスタンスがダウンしたため、アプリケーションにアクセスすると残り1つのインスタンスからのみレスポンスがあります。

Cloud Foundryは指定したインスタンス数を保つために自動でインスタンスを再起動させます。しばらくすると再び4つインスタンスからレスポンスがあることを確認できるでしょう。
