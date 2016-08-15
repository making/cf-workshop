## Pivotal Web Servicesのアカウント作成

本ワークショップではCloud Foundryのホスティングサービスとして[Pivotal Web Services](https://run.pivotal.io/) (PWS)を使用します。

### アカウントの作成

「SIGN UP FOR FREE」をクリック

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/38ab3187-0da5-f68a-8308-bac625543b00.png)

アカウント情報を入力

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/28668ed5-ec44-8b1f-1722-7a86af11a74e.png)

アクティベーション用リンクが入力したメールアドレスに送信されます

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/10934fc8-0109-7ff9-6da9-50d50921816a.png)

メールを確認し、「Verify your email address」をクリック

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/9004f268-c6b9-0cd1-62e0-e38f2ecad0a6.png)

「I Have read and agree to the Terms of Service for Pivotal Web Services」にチェックを入れ、「Next: Claim Your Trial」をクリック

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/680d2d0e-2cc7-7e04-6b0d-36e80e41b229.png)

電話番号を入力

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/7ff8e35d-8f8b-f939-b38a-025e94396834.png)

SMSにVerification Codeが送信されます

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/1ba72059-dcd8-7c9b-67c8-f5ac9e4a5f3a.png)

Verification Codeを入力して「Submit」をクリック

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/c8b45ddf-e8a6-e687-d0ce-c4e982a893f1.png)

Organization名(≒プロジェクト名)を入力し、「Start Free Trial」をクリック。
Organization名はイニシャルなどを使ってグローバルで一意の名前にしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/d5cec735-f3c3-2678-42bd-2e297d03ebf4.png)

コンソール画面が表示されます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/49270f3f-4f77-eae9-c1d7-e5e5e72371ca.png)

`cf login`コマンドでPWSにログインします。

``` console
$ cf login -a api.run.pivotal.io
API endpoint: api.run.pivotal.io

Email> yourmail@examle.com

Password>
Authenticating...
OK

Targeted org hajiboot

Targeted space development



API endpoint:   https://api.run.pivotal.io (API version: 2.58.0)   
User:           yourmail@example.com   
Org:            hajiboot
Space:          development
```

これでPWSにアクセスできます。
