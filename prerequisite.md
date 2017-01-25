


## 事前準備

### Java SE Development Kit 8

[Java SE Development Kit 8 Downloads](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)からJDKをインストールしてください。

環境変数`JAVA_HOME`にインストールしたディレクトリを設定してください。

Macユーザーは

``` console
export JAVA_HOME=`/usr/libexec/java_home`
```

で設定できます

### Curlコマンド

Macの場合はインストール不要です。

Windowsの場合は、[Git](https://git-scm.com/)をインストールすれば同梱されます。

### Cloud Foundry CLI


* [Windows 64 bit](https://cli.run.pivotal.io/stable?release=windows64&source=pws)
* [Windows 32 bit](https://cli.run.pivotal.io/stable?release=windows32&source=pws)
* [Mac OSX 64 bit](https://cli.run.pivotal.io/stable?release=macosx64&source=pws)
* [Linux 64 bit (.deb)](https://cli.run.pivotal.io/stable?release=debian64&source=pws)
* [Linux 32 bit (.deb)](https://cli.run.pivotal.io/stable?release=debian32&source=pws)
* [Linux 64 bit (.rpm)](https://cli.run.pivotal.io/stable?release=redhat64&source=pws)
* [Linux 32 bit (.rpm)](https://cli.run.pivotal.io/stable?release=redhat32&source=pws)

からインストーラーをダウンロードして`cf`コマンドをインストールしてください。

インストール後、`cf`にパスが通っていることを確認してください。

``` console
$ cf -v
cf version 6.15.0+fa1bfe2-2016-01-13
```

### Pivotal Web Servicesのアカウント作成

[こちら](pivotal-web-services.md)を参照してください。
