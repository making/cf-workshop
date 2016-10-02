## バックエンドサービスの利用 (Go編)

次はサービスを使ってみましょう。今回はサービスとしてRedisを使います。

### プロジェクト作成

先ほどの`hello-cf`とは別のアプリケーションを作成します。

以下のコマンドで雛形プロジェクトを作成してください。

``` console
$ mkdir -p $GOPATH/src/github.com/<your github account>/hello-redis
$ cd $GOPATH/src/github.com/<your github account>/hello-redis
```

[Glide](https://glide.sh)を使ってプロジェクトを初期化します。

``` console
$ glide init
[INFO] Generating a YAML configuration file and guessing the dependencies
[INFO] Attempting to import from other package managers (use --skip-import to skip)
```

Redisのライブラリである[`github.com/garyburd/redigo/redis`](https://github.com/garyburd/redigo)を`glide get`で追加します。

``` console
$ glide get github.com/garyburd/redigo/redis
[INFO] Preparing to install 1 package.
[INFO] Importing github.com/garyburd/redigo
[INFO] Downloading dependencies. Please wait...
[INFO] Fetching updates for github.com/garyburd/redigo.
[INFO] Resolving imports
[INFO] Downloading dependencies. Please wait...
```

`main.go`を作成して、次の内容を記述してください。

``` go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
	"time"

	"github.com/garyburd/redigo/redis"
)

func main() {

	c, err := redis.DialURL(redisURL())
	if err != nil {
		log.Fatal("Error! => ", err)
	}
	defer c.Close()

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		s, e := redis.String(c.Do("GET", "now"))
		if e != nil {
			now := time.Now().Format("2006-01-02T15:04:05Z0700")
			c.Do("SET", "now", now)
			s = now
		}
		fmt.Fprint(w, "Hello. It's "+s+" now")
	})
	http.ListenAndServe(":"+os.Getenv("PORT"), nil)
}

func redisURL() string {
	return "redis://localhost:6379" // Should be changed later
}
```

ローカル環境にRedisがインストールされている場合は、次のようにアプリケーションを起動して動作確認してください。


```
PORT=8080 go run main.go
```

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/92a4e074-20f8-bcb6-1a9a-31bdee645112.png)


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

サービスインスタンスをアプリケーションにバインドすると環境変数`VCAP_SERVICES`が`{"rediscloud":[{"credentials":{"hostname":"localhost","port":"6379","password":""},"name":"myredis"}]}`というJSON形式で渡されます。
そこでJSONをパースするライブラリを追加して、Redisサービスインスタンスへの接続情報を取得できるようにコードを変更しましょう。


JSONのライブラリである[`github.com/bitly/go-simplejson`](https://github.com/bitly/go-simplejson)を`glide get`で追加します。

``` console
$ glide get github.com/bitly/go-simplejson
[INFO] Preparing to install 1 package.
[INFO] Importing github.com/bitly/go-simplejson
[INFO] Downloading dependencies. Please wait...
[INFO] Fetching updates for github.com/garyburd/redigo.
[INFO] Fetching updates for github.com/bitly/go-simplejson.
[INFO] Resolving imports
[INFO] Downloading dependencies. Please wait...
```

次のように`go-simplejson`を使って`redisURL()`関数を変更します。

``` go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
	"time"

	"github.com/bitly/go-simplejson"
	"github.com/garyburd/redigo/redis"
)

func main() {

	c, err := redis.DialURL(redisURL())
	if err != nil {
		log.Fatal("Error! => ", err)
	}
	defer c.Close()

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		s, e := redis.String(c.Do("GET", "now"))
		if e != nil {
			now := time.Now().Format("2006-01-02T15:04:05Z0700")
			c.Do("SET", "now", now)
			s = now
		}
		fmt.Fprint(w, "Hello. It's "+s+" now")
	})
	http.ListenAndServe(":"+os.Getenv("PORT"), nil)
}

func redisURL() string {
	json, err := simplejson.NewJson([]byte(os.Getenv("VCAP_SERVICES")))
	if err != nil {
		log.Fatal("VCAP_SERVICES must be set!")
	}
	credentials := json.Get("rediscloud").GetIndex(0).Get("credentials") // PCFDevの場合はrediscloudではなくp-redis
	hostname := credentials.Get("hostname").MustString()
	port := credentials.Get("port").MustString()
	password := credentials.Get("password").MustString()
	return fmt.Sprintf("redis://:%s@%s:%s", password, hostname, port)
}
```


> **ノート**
> 
> ローカルで実行する場合は、次のように`VCAP_SERVICES`を定義してください。
>
> ```
> export VCAP_SERVICES='{"rediscloud":[{"credentials":{"hostname":"localhost","port":"6379","password":""},"name":"myredis"}]}'
> PORT=8080 go run main.go
> ```

それではアプリケーションをpushしましょう。アプリケーションの起動前にサービスインスタンスをバインドする必要があるため、いったん`--no-start`オプションをつけてpushします。

``` console
$ cf push hello-redis-tmaki -m 8m --no-start
Creating app hello-redis-tmaki in org APJ / space staging as tmaki@pivotal.io...
OK

Creating route hello-redis-tmaki.cfapps.io...
OK

Binding hello-redis-tmaki.cfapps.io to hello-redis-tmaki2...
OK

Uploading hello-redis-tmaki...
Uploading app files from: /Users/makit/go/src/github.com/making/hello-redis
Uploading 61.2K, 48 files
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

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/8ffd0c88-5fe1-0a2a-7b8b-8cb5667ebf97.png)



何回アクセスしてもキャッシュされているため、同じ結果が表示されることを確認してください。


### `go-cfenv`ライブラリを使う

`go-simplejson`でもサービスインスタンス情報にアクセスできますが、より便利なライブラリがCloud Foundry Communityから提供されています。
`go-cfenv`を使うとサービス名をハードコードしなくてもサービスインスタンス名やタグ名、ラベル名でサービスインスタンス情報にアクセスできます。


[`github.com/cloudfoundry-community/go-cfenv`](https://github.com/cloudfoundry-community/go-cfenv)を`glide get`で追加します。


``` console
$ glide get github.com/cloudfoundry-community/go-cfenv
[INFO] Preparing to install 1 package.
[INFO] Importing github.com/cloudfoundry-community/go-cfenv
[INFO] Downloading dependencies. Please wait...
[INFO] Fetching updates for github.com/bitly/go-simplejson.
[INFO] Fetching updates for github.com/cloudfoundry-community/go-cfenv.
[INFO] Fetching updates for github.com/garyburd/redigo.
[INFO] Resolving imports
[INFO] Downloading dependencies. Please wait...
```

`go-simplejson`は不要なのでremoveします。

``` console
$ glide rm github.com/bitly/go-simplejson
[INFO] Preparing to remove 1 packages.
[INFO] Resolving imports
[INFO] Package github.com/mitchellh/mapstructure found in vendor/ folder
[INFO] Package github.com/garyburd/redigo/internal found in vendor/ folder
```

`go-cfenv`を使うようにアプリケーションを修正します。

``` go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
	"time"

	"github.com/cloudfoundry-community/go-cfenv"
	"github.com/garyburd/redigo/redis"
)

func main() {
	c, err := redis.DialURL(redisURL())
	if err != nil {
		log.Fatal("Error! => ", err)
	}
	defer c.Close()

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		s, e := redis.String(c.Do("GET", "now"))
		if e != nil {
			now := time.Now().Format("2006-01-02T15:04:05Z0700")
			c.Do("SET", "now", now)
			s = now
		}
		fmt.Fprint(w, "Hello. It's "+s+" now")
	})
	http.ListenAndServe(":"+os.Getenv("PORT"), nil)
}

func redisURL() string {
	appEnv, err := cfenv.Current()
	if err != nil {
		log.Fatal("VCAP_SERVICES and VCAP_APPLICATION must be set!")
	}
	redis, err := appEnv.Services.WithName("myredis")
	if err != nil {
		log.Fatal(err)
	}
	hostname, _ := redis.CredentialString("hostname")
	port, _ := redis.CredentialString("port")
	password, _ := redis.CredentialString("password")
	return fmt.Sprintf("redis://:%s@%s:%s", password, hostname, port)
}
```

あとは`cf push`すれば良いです。

```
cf push hello-redis-tmaki
```

> **ノート**
> 
> ローカルで実行する場合は、次のように`VCAP_SERVICES`と`VCAP_APPLICATION`を定義してください。
>
> ```
> export VCAP_SERVICES='{"rediscloud":[{"credentials":{"hostname":"localhost","port":"6379","password":""},"name":"myredis"}]}'
> export VCAP_APPLICATION={}
> PORT=8080 go run main.go
> ```


### `manifest.yml`を用意する

次のような`manifest.yml`を作っておくと`cf push`だけでデプロイできて便利です。

``` yaml
applications:
- name: hello-tmaki-redis
  memory: 8m
  buildpack: go_buildpack
  services:
  - myredis
  env:
    GOVERSION: go1.7
```

この例ではGo 1.7を使用しています。

```
cf push
```
