## バックエンドサービスOracleの利用 (Java編) スタンドアローン版

次はサービスとしてOracle Databaseを使います。OracleデータベースのサービスはCloud Foundryのマーケットプレースには用意されていません。
この場合は、[Uer Provided Service](https://docs.cloudfoundry.org/devguide/services/user-provided.html)を使用してCloud Foundry外でプロビジョニングされたインスタンスをCloud Foundryのサービスインスタンスとして扱います。

### プロジェクト作成

新規にアプリケーションを作成します。

以下のコマンドで雛形プロジェクトを作成してください。


```
curl start.spring.io/starter.tgz \
       -d artifactId=hello-db \
       -d baseDir=hello-db \
       -d dependencies=data-rest,data-jpa,data-rest-hal,actuator \
       -d packageName=com.example \
       -d applicationName=HelloDbApplication | tar -xzvf -
```

#### Oracle JDBCドライバーの設定

Oracle Technology Networkから[ojdbc7.jar](http://www.oracle.com/technetwork/database/features/jdbc/jdbc-drivers-12c-download-1958347.html)をダウンロードしてください。この資料では`~/Downloads`にダウンロードしたこととします。

``` console
$ cd hello-db
$ mkdir -p vendor/com/oracle/ojdbc7/12.1.0.1.0
$ mv ~/Downloads/ojdbc7.jar vendor/com/oracle/ojdbc7/12.1.0.1.0/
```

ダウンロードしたOravle JDBCドライバーをローカルのMavenレポジトリにインストールするために`pom.xml`に次の内容を追記してください。

``` xml
<project>
<!-- ... -->
<!-- ここから -->
    <profiles>
        <profile>
            <id>default</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <dependencies>
                <dependency>
                    <groupId>com.oracle</groupId>
                    <artifactId>ojdbc7</artifactId>
                    <version>12.1.0.1.0</version>
                    <scope>runtime</scope>
                </dependency>
            </dependencies>
        </profile>
        <profile>
            <id>install-ojdbc</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-install-plugin</artifactId>
                        <executions>
                            <execution>
                                <id>install-ojdbc</id>
                                <phase>install</phase>
                                <goals>
                                    <goal>install-file</goal>
                                </goals>
                                <configuration>
                                    <file>
                                        ${basedir}/vendor/com/oracle/ojdbc7/12.1.0.1.0/ojdbc7.jar
                                    </file>
                                    <pomFile>
                                        ${basedir}/vendor/com/oracle/ojdbc7/12.1.0.1.0/ojdbc7-12.1.0.1.0.pom
                                    </pomFile>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
<!-- ここまで -->
</project>
```

`vendor/com/oracle/ojdbc7/12.1.0.1.0/ojdbc7-12.1.0.1.0.pom`を作成して、次の内容を記述してください。

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc7</artifactId>
    <version>12.1.0.1.0</version>
</project>
```

以下のコマンドを実行して作成することもできます。

``` bash
cat <<EOF > vendor/com/oracle/ojdbc7/12.1.0.1.0/ojdbc7-12.1.0.1.0.pom
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc7</artifactId>
    <version>12.1.0.1.0</version>
</project>
EOF
```

以下のコマンドを実行してOravle JDBCドライバーをローカルのMavenレポジトリにインストールしてください。

```
./mvnw install -P install-ojdbc -DskipTests=true
```

### ソースコードの追加

生成されたプロジェクトに以下の3ファイルを追加してください。

* `src/main/resources/application.properties`

``` properties
spring.jpa.hibernate.ddl-auto=update
# 以下、必要であればローカル開発用の設定
spring.datasource.url=jdbc:oracle:thin:@127.0.0.1:1521:orcl
spring.datasource.username=scott
spring.datasource.password=tiger
spring.datasource.driver-class-name=oracle.jdbc.driver.OracleDriver
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
    @GeneratedValue(strategy = GenerationType.AUTO)
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
cat <<EOF > src/main/resources/application.properties
spring.jpa.hibernate.ddl-auto=update
# spring.datasource.url=jdbc:oracle:thin:@127.0.0.1:1521:orcl
# spring.datasource.username=scott
# spring.datasource.password=tiger
# spring.datasource.driver-class-name=oracle.jdbc.driver.OracleDriver
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
    @GeneratedValue(strategy = GenerationType.AUTO)
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

### User Provided Serviceの作成

`cf create-user-provided-service <サービスインスタンス名> -p '<credentialsのJSON>'`で任意の`credentials`情報を設定できますが、
[Spring Cloud Connetors](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-cloud-foundry-connector.html#_oracle)のドキュメントを確認すると、`jdbcUrl`というフィールドにoracle形式のスキーマ文字列が設定されていれば自動で`DataSource`オブジェクトが作られるので、この形式を利用して次のUser Provided Serviceの作成を作成します。

```
$ cf create-user-provided-service mydb -p '{"jdbcUrl": "jdbc:oracle:thin:<User Name>/<Password>@<Host Name>:<Port>:<DB Name>"}'
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



次に`cf bind-service <App> <Service Instance>`コマンドでOracle DBサービスインスタンスをアプリケーションにバインドします。

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
  "user-provided": [
   {
    "credentials": {
     "jdbcUrl": "jdbc:oracle:thin:<User Name>/<Password>@<Host Name>:<Port>:<DB Name>"
    },
    "label": "user-provided",
    "name": "mydb",
    "syslog_drain_url": "",
    "tags": [],
    "volume_mounts": []
   }
  ]
 }
}

{
 "VCAP_APPLICATION": {
  "application_id": "8d59e897-67af-4c88-841f-5852c579ffd5",
  "application_name": "hello-db-tmaki",
  "application_uris": [
   "hello-db-tmaki.cfapps.io"
  ],
  "application_version": "8a4150f8-b232-4b3e-a17e-4577dae792a2",
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
  "version": "8a4150f8-b232-4b3e-a17e-4577dae792a2"
 }
}

No user-defined env variables have been set

No running env variables have been set

No staging env variables have been set
```

サービスインスタンスをバインドしたら`cf start`でアプリケーションを起動しましょう。

``` console
cf start hello-db-tmaki
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

> **Note**
> 
> 作成したアプリケーションにはHAL BrowserというREST APIの仕様や動作を確認するための画面が組み込まれています。 `http://hello-db-<your name>.cfapps.io/`にアクセスしてください。
> ![image](https://qiita-image-store.s3.amazonaws.com/0/1852/70d21411-d4d8-6447-c488-45f4b711dbf5.png)


アプリケーションにはOracleに関する設定を全く行いませんでしたが、何が起きているのでしょうか。`cf logs <アプリケーション名> --recent`コマンドでログを見てみましょう。

```
2016-11-07T11:11:00.45+0900 [APP/PROC/WEB/0]OUT 2016-11-07 02:11:00.457  INFO 20 --- [           main] urceCloudServiceBeanFactoryPostProcessor : Auto-reconfiguring beans of type javax.sql.DataSource
2016-11-07T11:11:00.50+0900 [APP/PROC/WEB/0]OUT 2016-11-07 02:11:00.503  INFO 20 --- [           main] o.c.r.o.s.c.s.r.PooledDataSourceCreator  : Found Tomcat high-performance connection pool on the classpath. Using it for DataSource connection pooling.
2016-11-07T11:11:00.53+0900 [APP/PROC/WEB/0]OUT 2016-11-07 02:11:00.538  INFO 20 --- [           main] urceCloudServiceBeanFactoryPostProcessor : Reconfigured bean dataSource into singleton service connector org.apache.tomcat.jdbc.pool.DataSource@6dc17b83{ConnectionPool[defaultAutoCommit=null; defaultReadOnly=null; defaultTransactionIsolation=-1; defaultCatalog=null; driverClassName=oracle.jdbc.OracleDriver; maxActive=4; maxIdle=100; minIdle=0; initialSize=0; maxWait=30000; testOnBorrow=true; testOnReturn=false; timeBetweenEvictionRunsMillis=5000; numTestsPerEvictionRun=0; minEvictableIdleTimeMillis=60000; testWhileIdle=false; testOnConnect=false; password=********; url=jdbc:oracle:thin:pcfuser/pcfpassword@pcfdemo.c5sgfwogyzmj.ap-northeast-1.rds.amazonaws.com:1521:PCFORCL; username=null; validationQuery=SELECT 'Y' from dual; validationQueryTimeout=-1; validatorClassName=null; validationInterval=3000; accessToUnderlyingConnectionAllowed=true; removeAbandoned=false; removeAbandonedTimeout=60; logAbandoned=false; connectionProperties=null; initSQL=null; jdbcInterceptors=null; jmxEnabled=true; fairQueue=true; useEquals=true; abandonWhenPercentageFull=0; maxAge=0; useLock=false; dataSource=null; dataSourceJNDI=null; suspectTimeout=0; alternateUsernameAllowed=false; commitOnReturn=false; rollbackOnReturn=false; useDisposableConnectionFacade=true; logValidationErrors=false; propagateInterruptState=false; ignoreExceptionOnPreLoad=false; }
```

`Auto-reconfiguring beans of type javax.sql.DataSource`という出力があり、バインドされたOracleサービスインスタンスへアクセスするためのデータソースが作成されていることがわかります。

Cloud Foundry環境ではステージング段階で用意された"Spring Auto Reconfiguration"が、アプリケーションにバインドされたサービスインスタンスを検出し、そのサービスインスタンスに対応したBean(ここでは`DataSource`)定義を作成してDIコンテナ中の既存の定義を書き換えます。 今回は環境変数`VCAP_SERVICES`中の`jdbcUrl`というフィールドにoracle形式のスキーマ文字列が設定されていることが[Oracleを検出するポイント](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-cloud-foundry-connector.html#_oracle)でした。

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
