## バックエンドサービスRedisの利用 (PHP編)

次はサービスを使ってみましょう。今回はサービスとしてRedisを使います。

### プロジェクト作成

先ほどの`hello-cf`とは別のディレクトリを作成します。

以下のコマンドで雛形プロジェクトを作成してください。

``` console
$ cd ..
$ mkdir hello-redis
$ cd hello-redis
```

`hello-redis`ディレクトリに`index.php`を作成して以下の内容を記述してください。

``` php
<?php
// Redisの接続情報を環境変数VCAP_SERVICESより取得
$vcap_services = json_decode(getenv('VCAP_SERVICES'));
$service_name = 'rediscloud'; // 次に選ぶサービス名
$service = $vcap_services->$service_name;
$credentials = $service[0]->credentials;

// Redisに接続
$redis = new Redis();
$redis->connect($credentials->hostname, $credentials->port);
$redis->auth($credentials->password);

// 接続結果を返す
echo $redis->ping();
```

次にPHPのRedis拡張モジュールを使用するために、`.bp-config`ディレクトリを作成してください。(**Windowsの場合はコマンドプロンプトやGit Bashなど、コマンドラインで作成してください。**)

``` console
$ mkdir .bp-config
```

`.bp-config`ディレクトリに`options.json`を作成して以下の内容を記述してください。

``` json
{
    "PHP_EXTENSIONS": [ "redis"]
}
```


### サービスを使う

Cloud Foundryのバックエンドサービスを使ってみましょう。`cf marketplace`コマンドで利用可能なサービス一覧を表示できます。

PWSの場合、次のサービスが用意されています。

``` console
$ cf marketplace
Getting services from marketplace in org tmaki / space development as ****@gmail.com...
OK

service          plans                                                                                description   
3scale           free_appdirect, basic_appdirect*, pro_appdirect*                                     API Management Platform   
app-autoscaler   bronze, gold                                                                         Scales bound applications in response to load (beta)   
blazemeter       free-tier, basic1kmr*, pro5kmr*                                                      Performance Testing Platform   
cedexisopenmix   opx_global*, openmix-gslb-with-fusion-feeds*                                         Openmix Global Cloud &amp; Data Center Load Balancer   
cedexisradar     free-community-edition                                                               Free Website&amp; Mobile App Performance Reports   
cleardb          spark, boost*, amp*, shock*                                                          Highly available MySQL for your Apps.   
cloudamqp        lemur, tiger*, bunny*, rabbit*, panda*                                               Managed HA RabbitMQ servers in the cloud   
cloudforge       free, standard*, pro*                                                                Development Tools In The Cloud   
elephantsql      turtle, panda*, hippo*, elephant*                                                    PostgreSQL as a Service   
flashreport      trial, basic*, silver*, gold*, platinum*                                             Generate PDF from your data   
ironworker       production*, starter*, developer*, lite                                              Job Scheduling and Processing   
loadimpact       lifree, li100*, li500*, li1000*                                                      Automated and on-demand performance testing   
memcachedcloud   100mb*, 250mb*, 500mb*, 1gb*, 2-5gb*, 5gb*, 30mb                                     Enterprise-Class Memcached for Developers   
memcachier       dev, 100*, 250*, 500*, 1000*, 2000*, 5000*, 7500*, 10000*, 20000*, 50000*, 100000*   The easiest, most advanced memcache.   
mongolab         sandbox                                                                              Fully-managed MongoDB-as-a-Service   
newrelic         standard                                                                             Manage and monitor your apps   
pubnub           free                                                                                 Build Realtime Apps that Scale   
rediscloud       100mb*, 250mb*, 500mb*, 1gb*, 2-5gb*, 5gb*, 10gb*, 50gb*, 30mb                       Enterprise-Class Redis for Developers   
searchify        small*, plus*, pro*                                                                  Custom search you control   
searchly         small*, micro*, professional*, advanced*, starter, business*, enterprise*            Search Made Simple. Powered-by Elasticsearch   
sendgrid         free, bronze*, silver*                                                               Email Delivery. Simplified.   
ssl              basic*                                                                               Upload your SSL certificate for your app(s) on your custom domain   
stamplay         plus*, premium*, core, starter*                                                      API-first development platform   
statica          starter, spike*, micro*, medium*, large*, enterprise*, premium*                      Enterprise Static IP Addresses   
temporize        small*, medium*, large*                                                              Simple and flexible job scheduling for your application   

* These service plans have an associated cost. Creating a service instance will incur this cost.

TIP:  Use 'cf marketplace -s SERVICE' to view descriptions of individual plans of a given service.
```

アプリケーションからサービスを使うには、まずはサービスインスタンスを作成する必要があります。
サービスインスタンスは`cf create-service <Service Name> <Plan> <Service Instance Name>`で作成できます。

Redisのサービスである`rediscloud`の`30mb`プラン（無償）のサービスインスタンスを`myredis`という名前で作成します。

``` console
$ cf create-service rediscloud 30mb myredis
```

`cf services`コマンドで作成したサービスインスタンス一覧を表示できます。

``` console
$ cf services
Getting services in org tmaki / space development as ****@gmail.com...
OK

name      service      plan   bound apps   last operation   
myredis   rediscloud   30mb                create succeeded 
```

アプリケーションをビルドでpushしましょう。アプリケーションの起動前にサービスインスタンスをバインドする必要があるため、いったん`--no-start`オプションをつけてpushします。
(**`cf push`は必ず`hello-redis`ディレクトリで行ってください**)

``` console
$ cf push hello-redis-tmaki -m 32m --no-start 
Creating app hello-redis-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK

Creating route hello-redis-tmaki.cfapps.io...
OK

Binding hello-redis-tmaki.cfapps.io to hello-redis-tmaki...
OK

Uploading hello-redis-tmaki...
Uploading app files from: /Users/makit/hello-redis
Uploading 415B, 1 files
Done uploading               
OK
```

次に`cf bind-service <App> <Service Instance>`コマンドで

``` console
$ cf bind-service hello-redis-tmaki myredis
Binding service myredis to app hello-redis-tmaki in org tmaki / space development as ****@gmail.com...
OK
TIP: Use 'cf restage hello-redis-tmaki' to ensure your env variable changes take effect
```

`cf env`でアプリケーションの環境変数を確認できます。環境変数`VCAP_SERVICES`にアプリケーションにバインドされたサービスインスタンスの情報がJSON形式で設定されます。

``` console
$ cf env hello-redis-tmaki
Getting env variables for app hello-redis-tmaki in org tmaki / space development as ****@gmail.com...
OK

System-Provided:
{
 "VCAP_SERVICES": {
  "rediscloud": [
   {
    "credentials": {
     "hostname": "pub-redis-13677.us-east-1-2.4.ec2.garantiadata.com",
     "password": "ChZds5T8YWxrK5Jx",
     "port": "13677"
    },
    "label": "rediscloud",
    "name": "myredis",
    "plan": "30mb",
    "provider": null,
    "syslog_drain_url": null,
    "tags": [
     "Data Stores",
     "Data Store",
     "Caching",
     "Messaging and Queuing",
     "key-value",
     "caching",
     "redis"
    ]
   }
  ]
 }
}

{
 "VCAP_APPLICATION": {
  "application_id": "a6512286-361c-4550-b21f-6fb6df6cf74d",
  "application_name": "hello-redis-tmaki",
  "application_uris": [
   "hello-redis-tmaki.cfapps.io"
  ],
  "application_version": "a70a2e02-c75b-4c1f-9c0c-c1013b46c7bd",
  "limits": {
   "disk": 1024,
   "fds": 16384,
   "mem": 1024
  },
  "name": "hello-redis-tmaki",
  "space_id": "1a199cc9-514a-42be-94eb-0e11b9b85ec6",
  "space_name": "development",
  "uris": [
   "hello-redis-tmaki.cfapps.io"
  ],
  "users": null,
  "version": "a70a2e02-c75b-4c1f-9c0c-c1013b46c7bd"
 }
}

No user-defined env variables have been set

No running env variables have been set

No staging env variables have been set
```

サービスインスタンスをバインドしたら`cf start`でアプリケーションを起動しましょう。

``` console
$ cf start hello-redis-tmaki
```

[https://hello-redis-tmaki.cfapps.io](https://hello-redis-tmaki.cfapps.io)にアクセスして、`+PONG`が出力され、Redisへの接続に成功していることを確認してください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/b40ed85f-f667-9ab0-b950-b972d13b3d75.png)

次にアプリケーションを少し変更して、データをキャッシュをしましょう。

``` php
<?php
// Redisの接続情報を環境変数VCAP_SERVICESより取得                                                                                                                                                                 
$vcap_services = json_decode(getenv('VCAP_SERVICES'));
$service_name = 'rediscloud';
$service = $vcap_services->$service_name;
$credentials = $service[0]->credentials;

// Redisに接続                                                                                                                                                                                                    
$redis = new Redis();
$redis->connect($credentials->hostname, $credentials->port);
$redis->auth($credentials->password);

// ここから変更
$now = $redis->get('time');
if (!$now) {
    $now = date('c');
    $redis->set('time', $now);
}

echo 'Hello. It\'s ' . $now . ' now';
```

再度、`cf push`してください。

``` console
$ cf push hello-redis-tmaki
```


![image](https://qiita-image-store.s3.amazonaws.com/0/1852/3f860de6-685d-437d-9291-184261df4fd2.png)

アプリケーションに何回アクセスしてもキャッシュされているため、同じ結果が表示されることを確認してください。


> **Note**
> 
> [cf-helper-php](https://github.com/cloudfoundry-community/cf-helper-php)は環境変数`VCAP_SERVICES`をパースして、代表的なサービス（RDB, Redis, MongoDB）への接続情報を簡単に作成できるライブラリです。
> このライブラリを使うと、ソースコードからサービス名(ここでは`rediscloud`)を隠蔽することができます。
