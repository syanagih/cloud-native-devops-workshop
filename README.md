![](common/images/customer.logo.png)
---
# ORACLE Cloud-Native DevOps workshop

## 説明

Oracle Cloud は、様々な業界で使用され、最も統合されている Public Cloud である。Oracle Cloud は、Software as a Service (SaaS), Platform as a Service (PaaS), そして、Infrastructure as a Service (IaaS) を跨って、また顧客のデータセンターに Oracle Cloud を配置して、最高クラスのサービスを提供している。Oracle Cloud は、ビジネスの俊敏性をあげ、コストを抑制し、また IT の複雑性を提言する事によりイノベーションやビジネスの変革の手助けをしている。このワークショップの内容は、様々なOracle Cloud Services 使い、Cloud 環境上におけるアプリケーション開発に関して様々な観点・考え方を紹介するものである。

### 前提

このワークショップは、Oracle PaaS トライアル・アカウントを保有して実施する事になっている。トライアル・アカウントを取得するためには、[トライアルの申請方法](common/request.for.trial.md) を参照して実施する。チュートリアルを実施するために、次の情報を確認しておく。そして、チュートリアルで必要になったら自身の情報で置き換えて実施する。

- Oracle Cloud アカウント **ユーザ名** と **パスワード**
- Oracle Cloud **アイデンティティ・ドメイン名**
- **データセンター / リージョン**

NOTE: 最初に Oracle Public Cloud Service を使い始める前に、あなたのアカウントにレプリケーション・ポリシーが設定されているかを確認しておく。もし設定されていない場合は、ほとんどのクラウド・サービスで必要になる Storage Cloud コンテナ を作成する事ができない。詳細は次のドキュメント: [Selecting a Replication Policy for Oracle Storage Cloud Service](https://docs.oracle.com/cloud/latest/storagecs_common/CSSTO/GUID-5D53C11F-3D9E-43E4-8D1D-DDBB95DEC715.htm) を参照。

### 重要事項

このワークショップの中で、パブリックなインターネット上で利用可能となるサービス・インスタンスをいくつか作成する事になる。これらのインスタンスが、たとえデモンストレーション目的であったとしても、オープンな環境で脆弱であったり一般的なパスワードを使わないというベスト・プラクティスを忘れてはいけない。そのため、このワークショップではどんなパスワードも指定しないので、自身でパスワードを定義する必要がある。チュートリアルの中でパスワードを求められる箇所があるが、後続の手順を実施のためにパスワードを覚えておく必要がある。

このワークショップは、Oracle Cloud 上でのアプリケーション開発の異なる観点毎に独立したいくつかのチュートリアルで成り立っている。これらのチュートリアルは、依存する前提事項がなければ、独立して実施する事ができる。

----

#### Developer Cloud Service, Application Container Cloud Service, Oracle Enterprise Pack For Eclipse を用いた Spring Boot アプリケーション開発ライフサイクルの支援

- [Spring Boot アプリケーション用のDeveloper Cloude Service プロジェクトの作成](springboot-sample/create.devcs.project.md)
- [Developer Cloud Service と Application Container Cloud Service を用いた継続的ビルド・インテグレーションの作成](springboot-sample/devcs.accs.ci.md)
- [Developer Cloud Service を用いた Eclipse 統合開発環境 (Oracle Enterprise Pack for Eclipse) の利用](oepe/setup.oepe.md)

#### Application Container Cloud Service 上の軽量コンテナで稼働するフロントエンド・アプリケーションと Java Cloud Service 上で稼働するバックエンド・リソースのバインド

- [Eclipse IDE (Oracle Enterprise Pack for Eclipse) と Developer Cloud Service を用いた開発ライフサイクル管理](devops-bind/README.md)

#### Java Mission Control と Java Flight Recorder の診断機能を用いた Application Container Cloud Service で稼働するアプリケーションのモニタリング

- [Application Conytainer Cloud Service 上のSpring Boot アプリケーションの監視及びチューニング](monitor-tune/README.md)

#### Integrate telemetry into continuous delivery and monitor an application using the Oracle Management Cloud
- [Deploying APM Agent on Apache Tomcat based application and setting up Application Performance Monitoring](apm/README.md)

#### Application Conainer Cloud Service での軽量 Java コンテナ(Tomcat) の実行

- [Tomcat ベースのアプリケーションを Application Container Cloud Service へデプロイ](accs-tomcat/README.md)
- [UI 及び PSM CLI を用いた Application Cloud Service のスケールアップ/スケールダウン](accs-psm/README.md)

#### Java Cloud Service への　Java EE アプリケーションのデプロイ

- [UI を用いた Database Cloud Service インスタンスの作成](dbcs-create/README.md)
- [UI を用いた Java Cloud Service インスタンスの作成](jcs-create/README.md)
- [TechCo (Java EE) サンプル・アプリケーション用 Database Cloud Service の準備](dbcs-prepare/README.md)
- [TechCo (Java EE) サンプル・アプリケーションの Java Cloud Service へのデプロイ](jcs-deploy/README.md)

#### UI と PaaS Service Manager を用いた Java Cloud Service の管理

- [Java Cloud Service のダイレクト・アクセス及び管理](jcs-direct/README.md)
- [UI を用いた Java Cloud Service クラスタのスケール・アウト](jcs-scale-ui/README.md)
- [PSM コマンド・ライン・インターフェースを用いた Java Cloud Service のスケール・イン](jcs-scale-psm/README.md)

#### 自動スケール・ポリシーによる Java Cloud Service のエラスティック・スケール

- [Java Cloud Service 自動スケール・ポリシー](jcs-autoscale/README.md)

#### オンプレミス WebLogic Server 11g (10.3.6) の 12cR2 マルチテナントへのアップグレード及び、Java Cloud Service への移行

- [DPCT (Domain to Partition Conversion Tool) を用いた WebLogic 11g ドメインから WebLogic 12cR2 ドメイン・パーティションへの変換](dpct/README.md)
+ [Move partition from WebLogic Server 12cR2 to Oracle Java Cloud Service](lift-and-shift/README.md)

####Migrate WebLogic 10.3.6 (on premise) Application to Java Cloud Service with App2Cloud tool ####

+ [Migrate Weblogic 10.3.6 (on premise) Application to Java Cloud Service with App2Cloud tool](app-2-cloud/README.md)

####Clean up the environment####

+ [Delete Java Cloud, Database Cloud and Database Container Services using user interface](cleanup/cleanup-ui.md)
+ [Delete Application Cloud Container Service using PaaS Service Manager (PSM) Command Line Interface (CLI)](cleanup/cleanup-psm.md)

---

####Customizing and personalizing the workshop content####

+ [Customize and personalize the workshop materials](customize/README.md)

---

## [Contributing](CONTRIBUTING.md)
Pull Requests are currently not being accepted. See [CONTRIBUTING](CONTRIBUTING.md) for details.

## [License](LICENSE.md)
Copyright (c) 2014, 2016 Oracle and/or its affiliates
The Universal Permissive License (UPL), Version 1.0
