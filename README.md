# cf-workshop

* [Introduction (今すぐ始めるCloud Foundry)](http://www.slideshare.net/makingx/cloud-foundry-hackt-hacktk) 🆕
* <s>[Introduction (Introduction to Cloud Foundry)](http://www.slideshare.net/makingx/introduction-to-cloud-foundry-jjug)</s>


## Contents

1. [事前準備 / Prerequisite](prerequisite.md)
1. 簡単なアプリケーションをデプロイ / Deploy hello world [[Java](deploy-application_java.md)] [[Go](deploy-application_go.md)] [[PHP](deploy-application_php.md)] [[PHP(簡易版)](deploy-application_php-simple.md)] [[Python](deploy-application_python.md)]
1. バックエンドサービス(Redis)の利用 / Use backend service(Redis) [[Java](backend-service-redis_java.md)] [[Go](backend-service-redis_go.md)] [[PHP(簡易版)](backend-service-redis_php-simple.md)] 
1. バックエンドサービス(MySQL)の利用 / Use backend service(MySQL) [[Java (JPA)](backend-service-mysql_java.md)] [[Java (MyBatis)](backend-service-mysql_java.md)]
1. バックエンドサービス(Oracle Database)の利用 / Use backend service(Oracle Database) [[Java](backend-service-oracle_java.md)]
1. スケールアウト / Scale out application [[Java](scale-out_java.md)] [[Go](scale-out_go.md)] [[PHP(簡易版)](scale-out_php-simple.md)]
1. [PCF Metricsによるアプリケーションのモニタリング / Application monitoring with PCF Metrics](pcf-metrics.md) 
1. [アプリケーションログの転送 / Forward application log](logging.md)
1. Blue-Greenデプロイ / Blue-Green deployment [[Java](blue-green-deployment_java.md)] [[Go](blue-green-deployment_go.md)] [[PHP(簡易版)](blue-green-deployment_php-simple.md)]
1. [PCF Devを用いたローカルCloud Foundry環境](pcf-dev.md)

To be continued

## Backlog

- [ ] One-Off Taskの実行
- [ ] Organization / Space / User / Roleの作成
- [ ] Auto Scaleの利用
- [x] User Provided Serviceの利用
- [x] PCF Devを使用したローカルCloud Foundry環境
- [x] Service Brokerの作成 ([別資料](https://github.com/Pivotal-Japan/service-broker-workshop))
- [x] PCF Metrixの利用
- [x] ログの転送にlogit.ioを追加
- [x] RabbitMQの利用([別資料](https://github.com/Pivotal-Japan/spring-cloud-stream-tutorial))
