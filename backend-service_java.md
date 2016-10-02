## バックエンドサービスの利用 (Java編)

次はサービスを使ってみましょう。今回はサービスとしてRedisを使います。

### プロジェクト作成

先ほどの`hello-cf`とは別のアプリケーションを作成します。

以下のコマンドで雛形プロジェクトを作成してください。

``` console
$ curl start.spring.io/starter.tgz \
       -d artifactId=hello-redis \
       -d baseDir=hello-redis \
       -d dependencies=web,actuator,cache,redis \
       -d applicationName=HelloRedisApplication | tar -xzvf -
```

生成されたプロジェクトの`hello-redis/src/main/java/com/example/HelloRedisApplication.java`を以下のように書き換えてください。

``` java
package com.example;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.OffsetDateTime;

@SpringBootApplication
@RestController
@EnableCaching // キャッシュ機能を有効にします
public class HelloRedisApplication {
	@Autowired
	Greeter greeter;

	@RequestMapping("/")
	String hello() {
		return greeter.hello();
	}

	public static void main(String[] args) {
		SpringApplication.run(HelloRedisApplication.class, args);
	}
}

@Component
class Greeter {
	@Cacheable("hello") // 実行結果をキャッシュします
	public String hello() {
		return "Hello. It's " + OffsetDateTime.now() + " now.";
	}
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

``` console
$ cd hello-redis
$ ./mvnw clean package
$ cf push hello-redis-tmaki -p target/hello-redis-0.0.1-SNAPSHOT.jar --no-start
Creating app hello-redis-tmaki in org tmaki / space development as ****@gmail.com...
OK

Creating route hello-redis-tmaki.cfapps.io...
OK

Binding hello-redis-tmaki.cfapps.io to hello-redis-tmaki...
OK

Uploading hello-redis-tmaki...
Uploading app files from: target/hello-redis-0.0.1-SNAPSHOT.jar
Uploading 498.8K, 91 files
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

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/5f9e014c-e422-6882-ba82-3a66f4c4462b.png)

何回アクセスしてもキャッシュされているため、同じ結果が表示されることを確認してください。

ここでは、Redisに関する設定を全く行いませんでしたが、何が起きているのでしょうか。ログを見てみましょう。

``` console
$ cf logs hello-redis-tmaki --recent
```

``` console
2016-03-17T22:19:50.04+0900 [APP/0]      OUT 2016-03-17 13:19:50.044  INFO 14 --- [           main] urceCloudServiceBeanFactoryPostProcessor : Auto-reconfiguring beans of type javax.sql.DataSource
2016-03-17T22:19:50.04+0900 [APP/0]      OUT 2016-03-17 13:19:50.047  INFO 14 --- [           main] urceCloudServiceBeanFactoryPostProcessor : No beans of type javax.sql.DataSource found. Skipping auto-reconfiguration.
2016-03-17T22:19:50.05+0900 [APP/0]      OUT 2016-03-17 13:19:50.051  INFO 14 --- [           main] edisCloudServiceBeanFactoryPostProcessor : Auto-reconfiguring beans of type org.springframework.data.redis.connection.RedisConnectionFactory
2016-03-17T22:19:50.11+0900 [APP/0]      OUT 2016-03-17 13:19:50.113  INFO 14 --- [           main] edisCloudServiceBeanFactoryPostProcessor : Reconfigured bean redisConnectionFactory into singleton service connector org.springframework.data.redis.connection.jedis.JedisConnectionFactory@51b6a3e3
```

`Reconfigured bean redisConnectionFactory`というメッセージが見えます。Java Buildpackに含まれるAuto Reconfigureという仕組みによって、サービスインスタンスのRedis情報から`RedisConnectionFactory` Beanを差し替えています。
これにより、アプリケーション側で特別な設定をすることなくCloud Foundry上でサービスインスタンスにアクセスすることができます。
ローカル開発時にローカル用Redisにアクセスする設定を行っている場合も、そのままCloud Foundryにデプロイして構いません。

Auto Reconfigureは**Springアプリケーション(とPlayアプリケーション)をデプロイした場合のみ有効**となる機能です。
[ドキュメント](https://github.com/cloudfoundry/java-buildpack-auto-reconfiguration#what-is-auto-reconfiguration)に記載されている通り、次のクラスのBeanが置換対象です。

* `javax.sql.DataSource`
* `org.springframework.amqp.rabbit.connection.ConnectionFactory`
* `org.springframework.data.mongodb.MongoDbFactory`
* `org.springframework.data.redis.connection.RedisConnectionFactory`
* `org.springframework.orm.hibernate3.AbstractSessionFactoryBean`
* `org.springframework.orm.hibernate4.LocalSessionFactoryBean`
* `org.springframework.orm.jpa.AbstractEntityManagerFactoryBean`

JSON内の`tags`の値やURLスキームを確認しています。

ただし、対象のBeanが複数定義ある場合(2つの`DataSource`など)や異なる同種サービスを同時に使う場合（MySQLとPostgreSQLなど）は、Beanの差し替えは発生しません。

Cloud Foundry (BuildPackの設定)でBeanのAuto Reconfigureされることにより接続先の情報が自動で設定され、今回のアプリケーションで使用しているSpring BootのAuto ConfigureによってRedisのキャッシュを使うための設定が自動化されています。

Spring Boot以外のアプリケーションを作成する場合は、環境変数`VCAP_SERVICES`に設定されているJSONをパースして接続先情報を取得します。`VCAP_SERVICES`の例を以下に示します。

``` json
{
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
```

#### (Advanced) Auto ReconfigureによるBeanの差し替えができない場合の対応

Auto ReconfigureによるBeanの差し替えができない場合は、手動でRedisの接続先情報を定義する必要があります。
Cloud Foundry上で自動で適用される`cloud`プロファイルにのみ利用されるように設定するのが良いです。

Javaで定義する場合は以下のようなコードを`HelloRedisApplication`クラス内に記述します。

``` java
@Profile("cloud")
@Bean
RedisProperties redisProperties(ObjectMapper objectMapper) throws IOException {
    JsonNode credentials = objectMapper.readTree(System.getenv("VCAP_SERVICES")).get("rediscloud").get(0).get("credentials");
    RedisProperties prop = new RedisProperties();
    prop.setHost(credentials.get("hostname").asText());
    prop.setPort(credentials.get("port").asInt());
    prop.setPassword(credentials.get("password").asText());
    return prop;
}
```

またはプロパティファイル(`src/main/resources/application-cloud.properties`)に以下の設定を行えばJavaコードを変更する修正はありません。

``` properties
spring.redis.host=${vcap.services.myredis.credentials.hostname}
spring.redis.port=${vcap.services.myredis.credentials.port}
spring.redis.password=${vcap.services.myredis.credentials.password}
```

他にも[Spring Cloud Connectors](http://cloud.spring.io/spring-cloud-connectors/)を使う方法もありますが、割愛します。[この記事](https://spring.io/blog/2015/04/27/binding-to-data-services-with-spring-boot-in-cloud-foundry)が詳しいです。

なお、Auto Reconfigureは以下のように環境変数を設定すれば無効化できます。

``` console
$ cf set-env hello-redis-tmaki JBP_CONFIG_SPRING_AUTO_RECONFIGURATION '{enabled: false}'
$ cf set-env hello-redis-tmaki SPRING_PROFILES_ACTIVE cloud # Auto Reconfigureを無効にするとcloudプロファイルも適用されなくなるため、cloudプロファイルは明示的に指定する。
$ cf restart hello-redis-tmaki
```
