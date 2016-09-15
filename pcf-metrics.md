### PCF Metricsによるアプリケーションのモニタリング

PWSの独自機能として、[PCF Metrics](https://metrics.run.pivotal.io)というアプリケーションのメトリクスやログを蓄積して視覚化する仕組みが用意されています。2016年8月時点でBETA版として無償で利用可能です。

PCF Metricsを利用することで直近1日間の

* コンテナのCPU、メモリ、ディスク使用率
* 単位秒あたりのリクエスト数、エラー数、リクエストのレイテンシー
* アプリケーションのイベント
* アプリケーションのログ

を確認することができます。


管理コンソールから「View in PCF Metrics」リンクをクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/215e8075-6064-47cf-c1f3-6792a73b5826.png)

**「PCF Metrics」へのリンク**

ダッシュボードへアクセスできます。ここではアプリケーションに発生したイベントと直近5分間のメトリクスを確認することができます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/06d03de0-a13c-a06a-1470-2c281f3aedbc.png)

**「PCF Metrics」のダッシュボード**

左側のメニューの「EXPLORE」をクリックしてください。ネットワークに関するメトリクスのグラフとアプリケーションログが表示されます。1分、1時間、1日という単位で状態を確認することができます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/1f46c343-2e1b-7577-0ff1-c12a19563417.png)

**ネットワークメトリクスとログ**

ログはキーワードで検索することもできます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/b9b6d9e2-3fce-48a8-d443-dea8a91b2fc3.png)

**ログの検索**

「Container Metrics」にチェックを入れることでコンテナに関するメトリクスのグラフも表示できます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/94dd63a3-2708-a84d-7ec2-05aab56a48d8.png)

**コンテナメトリクス**

「PCF Metrics」を利用することで直近でアプリケーションに何が起きたのか、どこでリクエストやCPU使用率のスパイクが発生したのかを把握することが可能になります。
アプリケーション開発者がこのような便利な機能をセットアップすることなく使えるようになるのはPaaSのメリットの一つです。

