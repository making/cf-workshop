## Pivotal Web Servicesのアカウント作成

本ワークショップではCloud Foundryのホスティングサービスとして[Pivotal Web Services](https://run.pivotal.io/) (PWS)を使用します。
本書ではPivotalが運用するパブリックなサービスである「Pivotal Web Services」(PWS)を使用します。
アカウント作成後、1年間$87分の無料期間(2GBメモリのアプリを2か月間利用する使用料に相当)が用意されています。

### アカウントの作成


まずは、アカウントを作成しましょう。

「[https://run.pivotal.io](https://run.pivotal.io)」 にアクセスし、「SIGN UP FOR FREE」をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/84f92baa-8ccb-4213-b22c-b925148a9673.png)

**「Pivotal Web Services」のトップ画面**

アカウント情報を入力してください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/28668ed5-ec44-8b1f-1722-7a86af11a74e.png)

**アカウント情報の入力**

アクティベーション用リンクが入力したメールアドレスに送信されます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/10934fc8-0109-7ff9-6da9-50d50921816a.png)

**アクティベーションメールの送信**

メールを確認し、「Verify your email address」をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/9004f268-c6b9-0cd1-62e0-e38f2ecad0a6.png)

**アクティベーションメールの確認**

「I Have read and agree to the Terms of Service for Pivotal Web Services」にチェックを入れ、「Next: Claim Your Trial」をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/680d2d0e-2cc7-7e04-6b0d-36e80e41b229.png)

**トライアルの開始**

電話番号を入力してください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/7ff8e35d-8f8b-f939-b38a-025e94396834.png)

**電話番号の入力**

SMSにVerification Codeが送信されます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/8a8be1c4-33b1-75dd-a455-5d12a0299d03.png)

**Verification Codeの確認**

Verification Codeを入力して「Submit」をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/c8b45ddf-e8a6-e687-d0ce-c4e982a893f1.png)

**Verification Codeの入力**

Organization名というプロジェクト名に相当する項目を入力し、「Start Free Trial」をクリックしてください。
Organization名はイニシャルなどを使ってグローバルで一意の名前にしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/d5cec735-f3c3-2678-42bd-2e297d03ebf4.png)

**Verification Codeの確認**

コンソール画面が表示されます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/49270f3f-4f77-eae9-c1d7-e5e5e72371ca.png)

**コンソール画面**


### 「Cloud Foundry CLI」のインストール

「Cloud Foundry」ではアプリケーションのデプロイや管理のための様々な操作にCLI(コマンドラインインターフェース)を使用します。

コンソール画面左の「Tools」リンクをクリックし、使用するOSを選択しダウンロードボタンをクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/4b9a136f-0327-4866-a199-2ba22dc5be1c.png)

**「Cloud Foundry CLI」のダウンロード**


ダウンロードしたインストーラーを開き、指示に従って操作し、 「Cloud Foundry CLI」のインストールしてください。

インストールが完了したら、ターミナルを立ち上げて、`cf -v`コマンドを実行してください。

**【ターミナル】 cfコマンドの確認**

``` console
$ cf -v
cf version 6.21.1+cd086c8-2016-08-18
```

### 「Pivotal Web Services」へログイン


`cf login`コマンドでPWSにログインできます。

**【ターミナル】 PWSへログイン**

``` console
$ cf login -a api.run.pivotal.io
API endpoint: api.run.pivotal.io

Email> メールアドレス

Password> パスワード
Authenticating...
OK

Targeted org hajiboot
Targeted space development

API endpoint:   https://api.run.pivotal.io (API version: 2.58.0)   
User:           メールアドレス   
Org:            hajiboot
Space:          development
```

これでPWSにアクセスできます。


> HTTP Proxyがある場合は、環境変数`https_proxy`を設定してください。[設定方法](https://docs.cloudfoundry.org/cf-cli/http-proxy.html)。
