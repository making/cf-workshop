## スケールアウト (Go編)

本章では[バックエンドサービスRedisの利用](backend-service-redis_go.md)で作成したプロジェクトを利用します。

Cloud Foundryではスケールアウトも簡単です。`cf scale -i <Instance Count> <App>`で指定したインスタンス数にスケールアウトできます。


``` console
$ cf scale -i 2 hello-redis-tmaki
```

ログを見て、2つ目のインスタンスであることを示す`[CELL/1]`が出力されていることを確認してください。

``` console
$ cf logs hello-redis-tmaki --recent
...
2016-10-06T17:53:57.67+0900 [CELL/1]     OUT Creating container
2016-10-06T17:54:05.72+0900 [CELL/1]     OUT Successfully created container
2016-10-06T17:54:06.41+0900 [CELL/1]     OUT Starting health monitoring of container
2016-10-06T17:54:08.50+0900 [CELL/1]     OUT Container became healthy
...
```

`cf apps`で対象のインスタンス数が`2/2`になっていることを確認できます。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name                requested state   instances   memory   disk   urls   
hello-redis-tmaki   started           2/2         8MB      1G     hello-redis-tmaki.cfapps.io
```

1インスタンスあたりのメモリは`cf scale -m <Memory <App>`で変更できます。

``` console
$ cf scale -m 32m hello-redis-tmaki
```

メモリを変更する場合はアプリケーションの再起動が必要な点に注意してください。

インスタンス数やメモリ数はPWSの管理画面からも変更できます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/fae8e9f5-10d9-9533-bcd7-620f6e912546.png)

Statusの表にインスタンスの情報が反映されます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/f705c419-76fc-6231-9f7f-355f951220c2.png)

Cloud Foundry内のRouterによってHTTPリクエストは2インスタンスに分散されますが、
インスタンス間でRedisキャッシュが共有されているため、出力結果は依然として同じです。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/400626da-2594-931f-fe6f-7c214c9428f0.png)


折角なので、どちらのインスタンスにアクセスしているか分かるようにソースコードを修正しましょう。

``` go
// 要 import "os"
fmt.Fprint(w, "Hello. It's "+s+" now ("+os.Getenv("CF_INSTANCE_INDEX")+")")
```

今度はインスタンス数2を指定して`cf push`します。

``` console
$ cf push hello-redis-tmaki -i 2
```

> 実は2回目以降の`cf push`は前回の情報が保存されているので`-i`は指定しなくても良いです

異なるインスタンスにアクセスしていることがわかります。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/32dce963-9182-ff9c-9ddb-68cc0d82b234.png)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/5e60401f-badb-5ae9-316d-ead34a0f6a96.png)


4インスタンスにスケールアウトしましょう。

``` console
$ cf scale -i 4 hello-redis-tmaki
```

しばらくすると3番目と4番目のインスタンスにもアクセスできていることがわかります。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/43108205-695e-c6b7-eb10-f1f48822fb8f.png)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/04f8b62e-0c97-4ab3-4597-24a914d4f2a3.png)


以下のコードを追加して、アプリケーションの強制シャットダウンできるようにしましょう。

``` go
	http.HandleFunc("/kill", func(res http.ResponseWriter, req *http.Request) {
		os.Exit(-1)
	})
```

再度pushしてください。


```
cf push hello-redis-tmaki
```

インスタンス0~3までアクセスできることを確認した後、
以下のコマンドを3回実行してください。

```
$ curl http://hello-redis-tmaki.cfapps.io/kill
502 Bad Gateway: Registered endpoint failed to handle the request.
```

4つのインスタンスのうち3つのインスタンスがダウンしたため、アプリケーションにアクセスすると残り1つのインスタンスからのみレスポンスがあります。

Cloud Foundryは指定したインスタンス数を保つために自動でインスタンスを再起動させます。しばらくすると再び4つインスタンスからレスポンスがあることを確認できるでしょう。
