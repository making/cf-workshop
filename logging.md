## アプリケーションログの転送

すでに見てきたように

``` console
$ cf logs <App>
```

でアプリケーションログを追跡できます。また、直近のログは

``` console
$ cf logs <App> --recent
```

で確認できます。

Cloud Foundryのアプリケーションログの保存は一時的ですが、過去のログを保管したり検索するために、外部のログマネージャーに転送できます。

本チュートリアルでは次の2つのサービスの使い方について見ていきます。

* [Logz.io](#logzioの場合)
* [Logit.io](#logitioの場合)
* [Papertrail](#papertrailの場合)

いずれか好きな方を選択してチュートリアルを進めてください。

### Logz.ioの場合

[Logz.io](https://logz.io/)のサービスを利用しましょう。

> Logz.io公式のCloud Foundryチュートリアルは[こちら](https://logz.io/blog/cloud-foundry-elk-stack/)("Shipping to Logz.io"以降)です。

"Enter Your Email"にメールアドレスを入力して、"Free Trial"をクリックしてください。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/1852/85b36c37-770b-90fd-9a51-454fc4e47344.png)

アカウント情報を入力して、"Start Your Free Trial"をクリックしてください。

ログインできたら、[Dashboard](https://app.logz.io)にアクセスして、バーの中央のの"Log Shippings"をクリックしてください。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/1852/f5b87bb7-f802-74f6-7af5-3c509a29b9fd.png)

左の"Platforms"メニューから"Applications"を開き、"[Cloud Foundry](https://app.logz.io/#/dashboard/data-sources/Cloud-Foundry)"を選択してください。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/1852/97832eb9-15a8-899b-683b-3a8042d28e0a.png)


[Step1]のテキストをコピーしてください。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/1852/fcf3776c-3cc4-2ce7-d353-648eac938731.png)

次のような形式になります。

```
cf cups my-log-drain -l https://listener.logz.io:8081?token=<TOKEN>
```

実行するとサービスインスタンスが作成され、`cf services`で指定したサービスインスタンス名が表示されることを確認できます。

``` console
$ cf services
Getting services in org tmaki / space development as ****@gmail.com...
OK

name            service         plan   bound apps          last operation   
myredis         rediscloud      30mb   hello-redis-tmaki   create succeeded   
my-log-drain    user-provided 
```

Redisの場合と同様にログ転送サービスインスタンスも`hello-redis-<your name>`にバインドしてrestartしてください。

```
$ cf bind-service hello-redis-tmaki my-log-drain
$ cf restart hello-redis-tmaki
```

これで`hello-redis-tmaki`のアプリケーションログはLogz.ioへ転送されます。

ダッシュボードからKibanaにアクセスしてください。虫眼鏡ボタンをクリックすると再検索されます。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/1852/f1db62f3-f6d2-56e1-6850-f6ae946e5e8b.png)


> 転送されるようになるまで、タイムラグがあります。Kibanaでログが表示されるようになるまでは、`cf restart`後もアプリケーションにアクセスし、ログを発生してください。

アプリケーションログの確認が終わったら`hello-redis-tmaki`を削除してください。

``` console
$ cf delete hello-redis-tmaki
```


### Logit.ioの場合



[Logit.io](https://logit.io/)のサービスを利用しましょう。

"Start your FREE trial now!"をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/cb33c12c-2d91-dcda-542e-f6afba68d1f5.png)

GithubかGoogleアカウントでログインしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/0915e3b9-ddf7-1a88-7a43-3dfc969a3a58.png)

"Is this a company account?"に「No」、"Daily Log Size"に「0.5」、"Retention"に「7」を入力して、"Continue"をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/e485470f-3041-701e-6358-632c20a1754e.png)

ページ上部の"Dashboard"をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/1fa3112c-78a9-a447-d2b0-0427d075ff98.png)

"Your stacks"のうち、Logstashの"Configuration"をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/4d330c27-cefb-fec3-7d0a-5081b26ba29b.png)

LogstashのURLとtcpのPORT番号をメモしてください。

<img width="80%" src="https://qiita-image-store.s3.amazonaws.com/0/1852/18b3c48c-9cc5-af23-4cea-24e14f48d2f3.png">

`cf create-user-provided-service <Service Instance Name> -l syslog://<URL:PORT>`を実行し、アプリケーションログ転送サービスインスタンスを作成します。

``` console
$ cf create-user-provided-service my-log-drain -l syslog://40da81a6-797b-4a3c-925c-8574d419e211-ls.logit.io:12394
Creating user provided service my-log-drain in org tmaki / space development as ****@gmail.com...
OK
```

`cf services`で指定したサービスインスタンス名が表示されることを確認できます。

``` console
$ cf services
Getting services in org tmaki / space development as ****@gmail.com...
OK

name            service         plan   bound apps          last operation   
myredis         rediscloud      30mb   hello-redis-tmaki   create succeeded   
my-log-drain    user-provided 
```

Redisの場合と同様にログ転送サービスインスタンスも`hello-redis-<your name>`にバインドしてrestartしてください。

```
$ cf bind-service hello-redis-tmaki my-log-drain
$ cf restart hello-redis-tmaki
```

これで`hello-redis-tmaki`のアプリケーションログはLogit.ioへ転送されます。


[Dashboard](https://logit.io/Dashboard)にアクセスし、Kibanaの"Access"をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/fa3f1dfa-8950-4832-feb0-ef9483da6ce2.png)

"Create"をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/07ab22d0-f735-6374-647f-8bf707549b29.png)

"Set as default index"をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/b0377176-59e1-bbd6-e145-2f2f24569452.png)

ページ上部の"Discover"をクリックすると、アプリケーションログを確認できます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/ac470807-cc17-357e-2343-98d21e9632d1.png)

アプリケーションログの確認が終わったら`hello-redis-tmaki`を削除してください。

``` console
$ cf delete hello-redis-tmaki
```


### Papertrailの場合

[Papertrail](https://papertrailapp.com/)のサービスを利用しましょう。

EmailとPasswordを入力して、「Start Logging - Free Plan」をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/39b90a9d-9cd1-bce8-d602-466a110ee104.png)

「Add your first system」をクリック

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/72aae222-8e71-772b-b292-f773604f5784.png)

「Other situations: Port 514 | Other」の「Other」をクリック

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/266426f4-f41e-8f9d-6e5d-811aa6abd070.png)

「I use Cloud Foundry」を選択し、「What should we call it?」にアプリケーション名を入力してください。ここでは`hello-cf`とします。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/946fc197-bbce-28e4-6414-aeb99db5ab4c.png)

表示された`logs*.papertrailapp.com:*****`をコピーしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/57c6c05c-c800-27ff-3641-590c8dc361a5.png)


`cf create-user-provided-service <Service Instance Name> -l syslog://<URL:PORT>`を実行し、アプリケーションログ転送サービスインスタンスを作成します。

``` console
$ cf create-user-provided-service my-log-drain -l syslog://logs4.papertrailapp.com:37190
Creating user provided service my-log-drain in org tmaki / space development as ****@gmail.com...
OK
```

`cf services`で指定したサービスインスタンス名が表示されることを確認できます。

``` console
$ cf services
Getting services in org tmaki / space development as ****@gmail.com...
OK

name            service         plan   bound apps          last operation   
myredis         rediscloud      30mb   hello-redis-tmaki   create succeeded   
my-log-drain    user-provided 
```

Redisの場合と同様にログ転送サービスインスタンスも`hello-redis-<your name>`にバインドしてrestartしてください。

```
$ cf bind-service hello-redis-tmaki my-log-drain
$ cf restart hello-redis-tmaki
```

これで`hello-redis-tmaki`のアプリケーションログはPapertrailへ転送されます。


[Dashboard](https://papertrailapp.com/dashboard)にアクセスし、`hello-cf`をクリックしてください。


![image](https://qiita-image-store.s3.amazonaws.com/0/1852/82857789-00a1-ab67-2e91-4ef40eb1771a.png)

アプリケーションログを確認できます。検索や絞り込みが可能です。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/38fb41e8-b183-7b8b-f466-2aa315f22b80.png)


アプリケーションログの確認が終わったら`hello-redis-tmaki`を削除してください。

``` console
$ cf delete hello-redis-tmaki
```



> **ノート**
> 
> アプリケーションログをFluentdに転送したい場合の設定は[こちら](http://docs.pivotal.io/pivotalcf/devguide/services/fluentd.html)を参照してください。
