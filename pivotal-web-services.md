## Pivotal Web Servicesのアカウント作成

本ワークショップではCloud Foundryのホスティングサービスとして[Pivotal Web Services](https://run.pivotal.io/) (PWS)を使用します。

### アカウントの作成

「SIGN UP FOR FREE」をクリック

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/38ab3187-0da5-f68a-8308-bac625543b00.png)

アカウント情報を入力

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/28668ed5-ec44-8b1f-1722-7a86af11a74e.png)

アクティベーション用リンクが入力したメールアドレスに送信されます

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/e48458b8-8169-92d0-af59-b2b811341b41.png)

メールを確認し、「Verify your email address」をクリック

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/abd00bb9-d275-80c6-5446-04d98eedd4a1.png)

「I Have read and agree to the Terms of Service for Pivotal Web Services」にチェックを入れ、「Next: Claim Your Trial」をクリック

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/2c50f0c3-c4dc-48ab-25f6-32016e10f90f.png)

電話番号を入力

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/15791603-49b7-b89d-5640-88dc39278b21.png)

SMSにVerification Codeが送信されます

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/1ba72059-dcd8-7c9b-67c8-f5ac9e4a5f3a.png)

Verification Codeを入力して「Submit」をクリック

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/c9539303-3398-e9ec-2a10-cbdeb9393552.png)

Organization名(≒プロジェクト名)を入力し、「Start 60-day Free Trial」をクリック。
Organization名はイニシャルなどを使って一意の名前にしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/137168/3c2b20bc-ee19-12ce-204c-e3e5b2400d64.png)

コンソール画面が表示されます。

![image](https://qiita-image-store.s3.amazonaws.com/0/137168/cb14e9a8-f649-c1d5-0898-9fad971cfac1.png)

`cf login`コマンドでPWSにログインします。

``` console
$ cf login -a api.run.pivotal.io
API endpoint: api.run.pivotal.io

Email> ***@gmail.com

Password>
Authenticating...
OK

Targeted org tmaki

Targeted space development



API endpoint:   https://api.run.pivotal.io (API version: 2.52.0)   
User:           ***@gmail.com   
Org:            tmaki   
Space:          development
```

これでPWSにアクセスできます。
