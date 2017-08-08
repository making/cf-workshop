## バックエンドサービスOracle Databaseの利用 (Java編, JPA版)

次はサービスとしてOracle Databaseを使います。OracleデータベースのサービスはCloud Foundryのマーケットプレースには用意されていません。
この場合は、[Uer Provided Service](https://docs.cloudfoundry.org/devguide/services/user-provided.html)を使用してCloud Foundry外でプロビジョニングされたインスタンスをCloud Foundryのサービスインスタンスとして扱います。

### プロジェクト作成

[バックエンドサービスMySQLの利用 (Java編)](backend-service-mysql_java.md)で作成したプロジェクトをそのまま利用します。
全章でデプロイしたアプリケーション、サービスインスタンスは削除してください。

```
cf d  -r -f hello-db-tmaki
cf ds -f mydb
```

#### MySQL JDBCドライバーの削除

`pom.xml`から`mysql:mysql-connector-java`の依存関係を削除してください。

``` xml
<!-- ここから

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>

ここまで削除 -->
```

#### Oracle JDBCドライバーの設定

Oracle Technology Networkから[ojdbc7.jar](http://www.oracle.com/technetwork/database/features/jdbc/jdbc-drivers-12c-download-1958347.html)をダウンロードしてください。この資料では`~/Downloads`にダウンロードしたこととします。

``` console
# hello-dbフォルダ内で
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

`vendor/com/oracle/ojdbc7/12.1.0.1.0/ojdbc7-12.1.0.1.0.pom`

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

以下のコマンドを実行してOravle JDBCドライバーをローカルのMavenレポジトリにインストールしてください。

```
$ ./mvnw install -P install-ojdbc -DskipTests=true
```

これでプロジェクトの設定は完了です。ソースコードに変更はありません。

### User Provided Serviceの作成

`cf create-user-provided-service <サービスインスタンス名> -p '<credentialsのJSON>'`で任意の`credentials`情報を設定できますが、
[Spring Cloud Connetors](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-cloud-foundry-connector.html#_oracle)のドキュメントを確認すると、Oracle Databaseにも対応しており、`jdbcUrl`というフィールドにoracle形式のスキーマ文字列が設定されていれば自動で`DataSource`オブジェクトが作られるので、この形式を利用して次のUser Provided Serviceの作成を作成します。

```
$ cf create-user-provided-service mydb -p '{"jdbcUrl": "jdbc:oracle:thin:<User Name>/<Password>@<Host Name>:<Port>:<DB Name>"}'
```



> **Note**
> 
> 開発環境にOracle DatabaseのVMを用意したい場合は
> http://www.oracle.com/technetwork/community/developer-vm/index.html#dbapp
> からVirtual Boxのイメージをダウンロードできます。ホストに1521番をポートフォワードしてください。

### アプリケーションのデプロイ

MySQLの時と全く同じようにアプリケーションをデプロイできます。

```
./mvnw clean package -DskipTests=true
cf push hello-db-tmaki -p target/hello-db-0.0.1-SNAPSHOT.jar --no-start
cf bind-service hello-db-tmaki mydb
cf start hello-db-tmaki
```

環境変数を確認してください。

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


### Spring Cloud Connectorsの利用

[MySQLの場合](https://github.com/Pivotal-Japan/cf-workshop/blob/master/backend-service-mysql_java.md#spring-cloud-connectorの利用)と全く同じです。
