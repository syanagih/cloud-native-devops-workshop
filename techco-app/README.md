![](../common/images/customer.logo.png)
---
# ORACLE Cloud-Native DevOps workshop
-----
## TechCo デモ・アプリケーションの Java Cloud Service へのデプロイ

### 説明

このチュートリアルでは、Database Cloud Service の準備と、WLST を用いての TechCo デモ・アプリケーションのデプロイを行う。

### チュートリアルについて

このチュートリアルは、スクリプトを用いた TechCo デモ・アプリケーションのデプロイを行うショートカット・手順でる。用意しているスクリプトは、以下の内容を実施する:

- Database Cloud Service へのユーザの作成とテーブルの作成
- Java Cloud Service へのリソースの作成
- Java Clode Service へのデモ・アプリケーションのデプロイ

### 前提

- このチュートリアルでは、オンプレミス環境のモデルの作成が必要である
  - [オンプレミス環境の準備](../common/vbox.vm.md) を参照
- TechCo デモ・アプリケーション用の準備を行うための Database Cloud Service インスタンス
  - チュートリアル:[UI を用いた Database Cloud Service インスタンスの作成](../dbcs-create/README.md) を参照
- TechCo デモ・アプリケーションのデプロイを行うための Java Cloud Service インスタンス
  - チュートリアル:[UI を用いた Java Cloud Service インスタンスの作成](../jcs-create/README.md) を参照

### 手順

#### SSH 秘密鍵のパスフレーズを SSH 認証エージェントに追加

SSH 秘密鍵のパスフレーズを複数回入力する作業を避けるために、秘密鍵とパスフレーズを SSH 認証エージェントに登録する必要がある。ターミナルを開き、`<クローンしたGitリポジトリ>/cloud-utils` へ移動する。そして秘密鍵のパスフレーズを SSH エージェントに追加を行う。

```
$ [oracle@localhost Desktop]$ cd /<クローンしたGitリポジトリ>/cloud-utils
$ [oracle@localhost cloud-utils]$ ssh-add pk.openssh
```

SSH 認証エージェントは秘密鍵がパスフレーズを持っている場合は訪ねてくる。***cloud-utils*** を使用して公開鍵と秘密鍵のペアを生成した場合は、`environment.properties` ファイル内の`ssh.passphrase` の値を確認する。
パスフレーズを入力する:

```
Enter passphrase for pk.openssh:
Identity added: pk.openssh (pk.openssh)
[oracle@localhost cloud.demos]$
```

アイデンティティ情報が追加されたので `ssh` や `scp` の実行中にパスフレーズを入力する必要がなくなった。

#### TechCo デモ・アプリケーションのデプロイ

Database Cloud Service、 Java Cloud Service 及び アプリケーションのデプロイメントがは全て単一のスクリプトで実施できる。`<クローンしたGitリポジトリ>/techco-app` に移動し、`deployTechCo.sh` を実行する:

```
$ [oracle@localhost Desktop]$ cd /u01/content/cloud-native-devops-workshop/techco-app
$ [oracle@localhost techco-app]$ ./deployTechCo.sh    
../cloud-utils/environment.properties found.
Identity domain: paasdemoXX
Java Cloud Service Public IP address: 129.144.18.102
Database Cloud Service Public IP address: 129.144.18.253
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building TechCo-ECommerce 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
...
...
...
SQL>
1 row created.

SQL>
1 row created.

SQL> SQL>
Commit complete.

SQL> Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
Database Cloud Service has been prepared.

Initializing WebLogic Scripting Tool (WLST) ...

Welcome to WebLogic Server Administration Scripting Shell

Type help() for help on available commands

Connecting to t3://winsdemoWLS-wls-1.compute-paasdemo16.oraclecloud.internal:7001 with userid weblogic ...

Successfully connected to Admin Server "winsdemo_adminserver" that belongs to domain "winsdemoWLS_domain".

Warning: An insecure protocol was used to connect to the server.
To ensure on-the-wire security, the SSL port or Admin port should be used instead.

Location changed to edit tree. 	 
This is a writable tree with DomainMBean as the root. 	 
To make changes you will need to start an edit session via startEdit().
For more help, use help('edit').

Starting an edit session ...
Started edit session, be sure to save and activate your changes once you are done.
Activating all your changes, this may take a while ...
The edit lock associated with this edit session is released once the activation is completed.
Activation completed
Deploying application from /tmp/TechCo-ECommerce-1.0-SNAPSHOT.war to targets winsdemoWLS_cluster (upload=false) ...
<Sep 14, 2016 10:25:09 PM UTC> <Info> <J2EE Deployment SPI> <BEA-260121> <Initiating deploy operation for application, TechCo-ECommerce-1.0-SNAPSHOT [archive: /tmp/TechCo-ECommerce-1.0-SNAPSHOT.war], to winsdemoWLS_cluster .>
..Completed the deployment of Application with status completed
Current Status of your Deployment:
Deployment command type: deploy
Deployment State : completed
Deployment Message : no message
Starting application TechCo-ECommerce-1.0-SNAPSHOT.
<Sep 14, 2016 10:25:18 PM UTC> <Info> <J2EE Deployment SPI> <BEA-260121> <Initiating start operation for application, TechCo-ECommerce-1.0-SNAPSHOT [archive: null], to winsdemoWLS_cluster .>
.Completed the start of Application with status completed
Current Status of your Deployment:
Deployment command type: start
Deployment State : completed
Deployment Message : no message
No stack trace available.

Exiting WebLogic Scripting Tool.

<Sep 14, 2016 10:25:22 PM UTC> <Warning> <JNDI> <BEA-050001> <WLContext.close() was called in a different thread than the one in which it was created.>
TechCo application has been deployed.
[oracle@localhost techco-app]$
```

最後にデプロイメント完了メッセージが確認できる。

#### デモ・アプリケーションの起動

Java Cloud Service にデプロイし起動したアプリケーションをテストするためには、ロードバランサを構成していないので、コンピュート・ノードのパブリック IP アドレスが必要になる。Java Cloud Service はインスタンスを生成するとパブリック IP アドレスが付与されるので、コンソール画面がから確認する。

または、***cloud-utils*** で提供しているツールを使用して、パブリック IP アドレス を REST API 経由で確認する事ができる。ターミナルを開き、`<クローンしたGitリポジトリ>/cloud-utils` へ移動し、次のコマンドを実施する:

```
$ [oracle@localhost Desktop]$ cd /u01/content/cloud-native-devops-workshop/cloud-utils
  $ [oracle@localhost cloud-utils]$ mvn -Dgoal=jcs-get-ip
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO]
[INFO] wins-cloud
[INFO] cloud-api
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building wins-cloud 1.0.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-install-plugin:2.4:install (default-install) @ wins-cloud ---
[INFO] Installing /u01/content/cloud-native-devops-workshop/cloud-utils/pom.xml to /home/oracle/.m2/repository/com/oracle/wins/cloud/wins-cloud/1.0.0-SNAPSHOT/wins-cloud-1.0.0-SNAPSHOT.pom
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building cloud-api 1.0.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ cloud-common ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /u01/content/cloud-native-devops-workshop/cloud-utils/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ cloud-common ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ cloud-common ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /u01/content/cloud-native-devops-workshop/cloud-utils/src/test/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ cloud-common ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ cloud-common ---
[INFO] Tests are skipped.
[INFO]
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ cloud-common ---
[INFO]
[INFO] --- maven-install-plugin:2.4:install (default-install) @ cloud-common ---
[INFO] Installing /u01/content/cloud-native-devops-workshop/cloud-utils/target/cloud-common.jar to /home/oracle/.m2/repository/com/oracle/wins/cloud/cloud-common/1.0.0-SNAPSHOT/cloud-common-1.0.0-SNAPSHOT.jar
[INFO] Installing /u01/content/cloud-native-devops-workshop/cloud-utils/pom.xml to /home/oracle/.m2/repository/com/oracle/wins/cloud/cloud-common/1.0.0-SNAPSHOT/cloud-common-1.0.0-SNAPSHOT.pom
[INFO]
[INFO] --- maven-antrun-plugin:1.8:run (first) @ cloud-common ---
[INFO] Executing tasks

main:
     [java] Read all properties from: file:/u01/content/cloud-native-devops-workshop/cloud-utils/environment.properties
     [java] Selected goal: jcs-get-ip
     [java] JCS get public IP address----------------------------------------
     [java] Executing request GET http://jaas.oraclecloud.com/paas/service/jcs/api/v1.1/instances/paasdemo16/winsdemoWLS HTTP/1.1 Auth: {<any realm>@jaas.oraclecloud.com:443=[principal: demouser]}
     [java] Response: HTTP/1.1 200 OK
     [java] Public IP address of the JCS instance: 129.144.18.102
[INFO] Executed tasks
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO]
[INFO] wins-cloud ......................................... SUCCESS [  0.614 s]
[INFO] cloud-api .......................................... SUCCESS [  6.488 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 7.379 s
[INFO] Finished at: 2016-09-14T15:42:49-07:00
[INFO] Final Memory: 12M/491M
[INFO] ------------------------------------------------------------------------
[oracle@localhost cloud-utils]$
```

ブラウザを開き、Java Cloud Service のパブリック IP アドレス を用いて以下の　URL を開く:

- `https://<JCS パブリック IP アドレス>/TechCo-ECommerce`

デモ・アプリケーション画面を確認する事ができる。

![](../jcs-deploy/images/28.png)

Java Cloud Service が ロードバランサを構成している場合は、パブリック IP アドレスは、ロードバランサのアドレスを使用する事で、デモ・アプリケーションにアクセスする事が可能である。
