## ログの転送

すでに見てきたように

``` console
$ cf logs <App>
```

でログを追跡できます。また、直近のログは

``` console
$ cf logs <App> --recent
```

で確認できます。

Cloud Foundryのログの保存は一時的ですが、過去のログを保管したり検索するために、外部のログマネージャーに転送できます。

ここでは[Papertrail](https://papertrailapp.com/)のサービスを利用しましょう。

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


`cf create-user-provided-service <Service Instance Name> -l syslog://<URL:PORT>`を実行し、ログ転送サービスインスタンスを作成します。

``` console
$ cf create-user-provided-service papertrail -l syslog://logs4.papertrailapp.com:37190
Creating user provided service papertrail in org tmaki / space development as ****@gmail.com...
OK
```

`cf services`で指定したサービスインスタンス名が表示されることを確認できます。

``` console
$ cf services
Getting services in org tmaki / space development as ****@gmail.com...
OK

name         service         plan   bound apps          last operation   
myredis      rediscloud      30mb   hello-redis-tmaki   create succeeded   
papertrail   user-provided 
```

Redisの場合と同様にログ転送サービスインスタンスも`hello-redis-<your name>`にバインドしてrestartしてください。

```
$ cf bind-service hello-redis-tmaki papertrail
$ cf restart hello-redis-tmaki
```

これで`hello-redis-tmaki`のログはPapertrailへ転送されます。

> 本チュートリアルを[PCF Dev](pcf-dev.md)で実施する場合は、[v0.16.0](https://github.com/pivotal-cf/pcfdev/releases/tag/v0.16.0)以上を使用してください

[Dashboard](https://papertrailapp.com/dashboard)にアクセスし、`hello-cf`をクリックしてください。


![image](https://qiita-image-store.s3.amazonaws.com/0/1852/82857789-00a1-ab67-2e91-4ef40eb1771a.png)

ログを確認できます。検索や絞り込みが可能です。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/38fb41e8-b183-7b8b-f466-2aa315f22b80.png)


ログの確認が終わったら`hello-redis-tmaki`を削除してください。

``` console
$ cf delete hello-redis-tmaki
```

