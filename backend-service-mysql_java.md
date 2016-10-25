## バックエンドサービスMySQLの利用 (Java編)

次はサービスとしてMySQLを使います。

### プロジェクト作成

新規にアプリケーションを作成します。

以下のコマンドで雛形プロジェクトを作成してください。


```
curl start.spring.io/starter.tgz \
       -d artifactId=hello-db \
       -d baseDir=hello-db \
       -d dependencies=data-rest,data-jpa,data-rest-hal,actuator,mysql \
       -d applicationName=HelloDbApplication | tar -xzvf -
```

生成されたプロジェクトに以下の3ファイルを追加してください。

* `src/main/resources/application.properties`

``` properties
spring.jpa.hibernate.ddl-auto=update
# 以下、必要であればローカル開発用の設定
spring.datasource.url=jdbc:mysql://localhost:3306/demo
spring.datasource.username=root
spring.datasource.password=
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

* `src/main/java/com/example/Message.java`

``` java
package com.example;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Message {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    public Long id;
    public String text;
}
```

* `src/main/java/com/example/MessageRepository.java`

``` java
package com.example;

import org.springframework.data.repository.CrudRepository;

public interface MessageRepository extends CrudRepository<Message, Long> {
}
```

以下、自動化スクリプトです。

``` bash
cd hello-db
cat <<EOF > src/main/resources/application.properties
spring.jpa.hibernate.ddl-auto=update
# spring.datasource.url=jdbc:mysql://localhost:3306/demo
# spring.datasource.username=root
# spring.datasource.password=
# spring.datasource.driver-class-name=com.mysql.jdbc.Driver
EOF
cat <<EOF > src/main/java/com/example/Message.java
package com.example;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Message {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    public Long id;
    public String text;
}
EOF
cat <<EOF > src/main/java/com/example/MessageRepository.java
package com.example;

import org.springframework.data.repository.CrudRepository;

public interface MessageRepository extends CrudRepository<Message, Long> {
}
EOF
``` 

たったこれだけでmessageテーブルのCRUD処理するためのREST APIができました。

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

MySQLのサービスである`cleardb`の`spark`プラン（無償）のサービスインスタンスを`mydb`という名前で作成します。

``` console
$ cf create-service cleardb spark mydb
```

`cf services`コマンドで作成したサービスインスタンス一覧を表示できます。

``` console
$ cf services
Getting services in org APJ / space staging as tmaki@example.com...
OK

name                      service         plan      bound apps     last operation
mydb                      cleardb         spark                    create succeeded
```

アプリケーションをビルドでpushしましょう。アプリケーションの起動前にサービスインスタンスをバインドする必要があるため、いったん`--no-start`オプションをつけてpushします。


``` console
$ ./mvnw clean package -DskipTests=true
$ cf push hello-db-tmaki -p target/hello-db-0.0.1-SNAPSHOT.jar --no-start
Creating app hello-db-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK

Creating route hello-db-tmaki.cfapps.io...
OK

Binding hello-db-tmaki.cfapps.io to hello-db-tmaki...
OK

Uploading hello-db-tmaki...
Uploading app files from: /var/folders/15/fww24j3d7pg9sz196cxv_6xm4nvlh8/T/unzipped-app028999524
Uploading 461.1K, 99 files
Done uploading               
OK
```


``` console
$ cf create-service cleardb spark mydb
```

```
$ cf services
Getting services in org APJ / space staging as tmaki@example.com...
OK

name                      service         plan      bound apps     last operation
mydb                      cleardb         spark                    create succeeded
```

```
$ cf push hello-db-tmaki -p target/hello-db-0.0.1-SNAPSHOT.jar --no-start
Creating app hello-db-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK

Creating route hello-db-tmaki.cfapps.io...
OK

Binding hello-db-tmaki.cfapps.io to hello-db-tmaki...
OK

Uploading hello-db-tmaki...
Uploading app files from: /var/folders/15/fww24j3d7pg9sz196cxv_6xm4nvlh8/T/unzipped-app028999524
Uploading 461.1K, 99 files
Done uploading               
OK

```


次に`cf bind-service <App> <Service Instance>`コマンドで

``` console
$ cf bind-service hello-db-tmaki mydb
Binding service mydb to app hello-db-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK
TIP: Use 'cf restage hello-db-tmaki' to ensure your env variable changes take effect
```

`cf env`でアプリケーションの環境変数を確認できます。環境変数`VCAP_SERVICES`にアプリケーションにバインドされたサービスインスタンスの情報がJSON形式で設定されます。

``` console
$ cf env hello-db-tmaki
Getting env variables for app hello-db-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK

System-Provided:
{
 "VCAP_SERVICES": {
  "cleardb": [
   {
    "credentials": {
     "hostname": "us-cdbr-iron-east-04.cleardb.net",
     "jdbcUrl": "jdbc:mysql://us-cdbr-iron-east-04.cleardb.net/ad_ce89352eddfecd1?user=bef832bc83cc21\u0026password=c87745ab",
     "name": "ad_ce89352eddfecd1",
     "password": "c77f45ab",
     "port": "3306",
     "uri": "mysql://bef832bc83cc21:c87f45ab@us-cdbr-iron-east-04.cleardb.net:3306/ad_ce99352eddfecd1?reconnect=true",
     "username": "bef832bc83cc21"
    },
    "label": "cleardb",
    "name": "mydb",
    "plan": "spark",
    "provider": null,
    "syslog_drain_url": null,
    "tags": [
     "Cloud Databases",
     "Data Stores",
     "Web-based",
     "Online Backup \u0026 Storage",
     "Single Sign-On",
     "Cloud Security and Monitoring",
     "Certified Applications",
     "Developer Tools",
     "Data Store",
     "Development and Test Tools",
     "Buyable",
     "mysql",
     "relational"
    ],
    "volume_mounts": []
   }
  ]
 }
}

{
 "VCAP_APPLICATION": {
  "application_id": "2759c9d0-582b-4cfc-86d3-725ae3535b33",
  "application_name": "hello-db-tmaki",
  "application_uris": [
   "hello-db-tmaki.cfapps.io"
  ],
  "application_version": "a2500706-74f5-43d0-a5ea-8df139b39d54",
  "limits": {
   "disk": 1024,
   "fds": 16384,
   "mem": 1024
  },
  "name": "hello-db-tmaki",
  "space_id": "82cdf564-7628-41a0-8595-870ee2cd26bc",
  "space_name": "staging",
  "uris": [
   "hello-db-tmaki.cfapps.io"
  ],
  "users": null,
  "version": "a2500706-74f5-43d0-a5ea-8df139b39d54"
 }
}

No user-defined env variables have been set

No running env variables have been set

No staging env variables have been set
```

サービスインスタンスをバインドしたら`cf start`でアプリケーションを起動しましょう。

``` console
$ cf start hello-db-tmaki
```


`curl`コマンドでmessage一覧を取得してください。

``` console
$ curl http://hello-db-tmaki.cfapps.io/messages
{
  "_embedded" : {
    "messages" : [ ]
  },
  "_links" : {
    "self" : {
      "href" : "http://hello-db-tmaki.cfapps.io/messages"
    },
    "profile" : {
      "href" : "http://hello-db-tmaki.cfapps.io/profile/messages"
    }
  }
}
```

次にmessageにデータを登録してください。

``` console
$ curl -H 'Content-Type: application/json' -d '{"text":"Hello World!"}' http://hello-db-tmaki.cfapps.io/messages
{
  "text" : "Hello World!",
  "_links" : {
    "self" : {
      "href" : "http://hello-db-tmaki.cfapps.io/messages/2"
    },
    "message" : {
      "href" : "http://hello-db-tmaki.cfapps.io/messages/2"
    }
  }
}
```

登録したデータを確認してください。

``` console
$ curl http://hello-db-tmaki.cfapps.io/messages
{
  "_embedded" : {
    "messages" : [ {
      "text" : "Hello World!",
      "_links" : {
        "self" : {
          "href" : "http://hello-db-tmaki.cfapps.io/messages/2"
        },
        "message" : {
          "href" : "http://hello-db-tmaki.cfapps.io/messages/2"
        }
      }
    } ]
  },
  "_links" : {
    "self" : {
      "href" : "http://hello-db-tmaki.cfapps.io/messages"
    },
    "profile" : {
      "href" : "http://hello-db-tmaki.cfapps.io/profile/messages"
    }
  }
}
```

登録したデータを削除してください。

``` console
$ curl -XDELETE http://hello-db-tmaki.cfapps.io/messages/2
```

アプリケーションにはMySQLに関する設定を全く行いませんでしたが、何が起きているのでしょうか。`cf logs <アプリケーション名> --recent`コマンドでログを見てみましょう。

```
2016-08-21T23:36:44.10+0900 [APP/0]      OUT 2016-08-21 14:36:44.106  INFO 28 --- [           main] urceCloudServiceBeanFactoryPostProcessor : Auto-reconfiguring beans of type javax.sql.DataSource
2016-08-21T23:36:44.12+0900 [APP/0]      OUT 2016-08-21 14:36:44.124  INFO 28 --- [           main] o.c.r.o.s.c.s.r.PooledDataSourceCreator  : Found Tomcat high-performance connection pool on the classpath. Using it for DataSource connection pooling.
2016-08-21T23:36:44.18+0900 [APP/0]      OUT 2016-08-21 14:36:44.180  INFO 28 --- [           main] urceCloudServiceBeanFactoryPostProcessor : Reconfigured bean dataSource into singleton service connector org.apache.tomcat.jdbc.pool.DataSource@646be2c3{ConnectionPool[defaultAutoCommit=null; defaultReadOnly=null; defaultTransactionIsolation=-1; defaultCatalog=null; driverClassName=org.mariadb.jdbc.Driver; maxActive=4; maxIdle=100; minIdle=0; initialSize=0; maxWait=30000; testOnBorrow=true; testOnReturn=false; timeBetweenEvictionRunsMillis=5000; numTestsPerEvictionRun=0; minEvictableIdleTimeMillis=60000; testWhileIdle=false; testOnConnect=false; password=********; url=jdbc:mysql://us-cdbr-iron-east-04.cleardb.net/ad_65534bc92cbcbcc?user=b568f82bc8e0a0&password=7b4cee48; username=null; validationQuery=/* ping */ SELECT 1; validationQueryTimeout=-1; validatorClassName=null; validationInterval=30000; accessToUnderlyingConnectionAllowed=true; removeAbandoned=false; removeAbandonedTimeout=60; logAbandoned=false; connectionProperties=null; initSQL=null; jdbcInterceptors=null; jmxEnabled=true; fairQueue=true; useEquals=true; abandonWhenPercentageFull=0; maxAge=0; useLock=false; dataSource=null; dataSourceJNDI=null; suspectTimeout=0; alternateUsernameAllowed=false; commitOnReturn=false; rollbackOnReturn=false; useDisposableConnectionFacade=true; logValidationErrors=false; propagateInterruptState=false; ignoreExceptionOnPreLoad=false; }
```

`Auto-reconfiguring beans of type javax.sql.DataSource`という出力があり、バインドされたMySQLサービスインスタンスへアクセスするためのデータソースが作成されていることがわかります。

Cloud Foundry環境ではステージング段階で用意された"Spring Auto Reconfiguration"が、アプリケーションにバインドされたサービスインスタンスを検出し、そのサービスインスタンスに対応したBean(ここでは`DataSource`)定義を作成してDIコンテナ中の既存の定義を書き換えます。 今回は環境変数`VCAP_SERVICES`中の`tags`キーに`mysql`が含まれていることが[MySQLを検出するポイント](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-cloud-foundry-connector.html#_mysql)でした。

これによりローカル環境向けに作成した実行可能jarファイルをそのままCloud Foundryにデプロイしてバックエンドサービスを使うこともできるため、設定を減らすことができます。

バックエンドサービスを使うBean定義を明示的に行いたい場合は[Spring Cloud Connector](http://cloud.spring.io/spring-cloud-connectors/)を利用すると便利です。

### Spring Cloud Connectorの利用

Spring Auto Reconfigureは便利ですが、コネクションプールの接続数やタイムアウトの設定ができません。次のSpring Cloud Connectorを利用すると環境変数`VCAP_SERVICES`から接続情報を取得してデータアクセス必要なオブジェクト（RDBMSの場合は`DataSource`、Redisの場合は`RedisConnectionFactory`）

``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-spring-service-connector</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-cloudfoundry-connector</artifactId>
</dependency>
```

次にSpring Cloud Connectorsを使用した次のJavaConfigを作成しましょう。


``` java
package com.example;

import javax.sql.DataSource;

import org.springframework.cloud.config.java.AbstractCloudConfig;
import org.springframework.cloud.service.PooledServiceConnectorConfig;
import org.springframework.cloud.service.relational.DataSourceConfig;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
@Profile("cloud")
public class CloudConfig extends AbstractCloudConfig {
	@Bean
	DataSource dataSource() {
		PooledServiceConnectorConfig.PoolConfig poolConfig = new PooledServiceConnectorConfig.PoolConfig(
				5 /* min */, 30 /* max */, 3000 /* maxWait */);
		return connectionFactory().dataSource(new DataSourceConfig(poolConfig, null));
	}
}
```

`DataSource`の詳細な設定方法は[ドキュメント](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-spring-service-connector.html#_relational_database_db2_mysql_oracle_postgresql_sql_server)を参照してください。

Spring Cloud Connectorsを使用する場合は**Auto Reconfigureは無効になります**。

> **WARNING**
> 
> `DataSource`のAuto-Reconfigurationを利用すると次のログが出力されます。
> 
> ```
> org.apache.tomcat.jdbc.pool.ConnectionPool         WARNING maxIdle is larger than maxActive, setting maxIdle to: 4
> ```
> 
> これはAuto-Reconfiguration側で最大接続数を`4`に指定しているからです。(バックエンドサービスの無償枠向け)
> 
> https://discuss.pivotal.io/hc/en-us/articles/221898227-Connection-pool-warning-message-maxIdle-is-larger-than-maxActive-setting-maxIdle-to-4-seen-in-PCF-deployed-Spring-app
> 
> 
> 基本的には`spring-cloud-connector`を使って、コネクションプールの設定をすべきです。
