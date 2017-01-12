![](../common/images/customer.logo.png)
---
# ORACLE Cloud-Native DevOps workshop #
----
## オンプレミス WebLogic Server 10.3.6 アプリケーションの Java Cloud Service 移行

### チュートリアルについて

このチュートリアルによるユースケースは、オンプレミスにWebLogic Server 10.3.6 上で稼働している Java EE 5 アプリケーション持っており、それをパブリッククラウドに移行したいというもので、その場合に Java Cloud Service の **AppToCloud** 機能を使用して移行が行えるというものである。AppToCloud は、既存の WebLogic ドメイン構成とアプリケーションをエクスポートし、それを用いて同じリソース、同じアプリケーションを持つ同じドメインによる Java Cloud Service インスタンスを作成する事ができる。以下オンプレミスとクラウド移行後のアーキテクチャを示す図である。移行処理は、完全に AppToCloud 機能による実施され、クラウドへの移行を最低限の干渉で実現するものである。

![](images/app2cloud.arhitecture.png)

このチュートリアルは、以下を実施する:

- オンプレミス WebLogic 10.3.6 ドメインを App2ToCloud によりアプリケーションを含んだ状態でエクスポートする
- AppToCloud により Java Cloud Service にインスタンスを作成する
- オンプレミスの WebLogic 10.3.6 で稼働しエクスポートしたアプリケーションをインポートする

### 前提

- チュートリアル用の VirtualBox イメージ、又は[カスタム環境を用意](../common/vbox.vm.md)する事
- Java Cloud Service が依存しない空の Database Cloud Service インスタンスが稼働している事

### 手順

#### WebLogic Server 10.3.6 ドメインの作成と Petstore デモ・アプリケーションのデプロイ

ターミナル・ウインドウを開き、`<クローンしたGitリポジトリ>/app-2-cloud` に移動する。

```
$ [oracle@localhost Desktop]$ cd /u01/content/cloud-native-devops-workshop/app-2-cloud
```

`prepareEnv.sh` を実行する。これにより、データベースが起動し、WebLogic Server 10.3.6 ドメインが作成され、WebLogic インスタンスが起動し、Petstore デモ・アプリケーションがデプロイされる。このスクリプトの使用方法は次のようになる:

- `prepareEnv.sh <DBユーザ名> <DBパスワード> [<PDB名>]`

以下、VirtualBox 環境での実行例である:

```
$ [oracle@localhost app-2-cloud]$ ./prepareEnv.sh system welcome1
Oracle database (sid: orcl) is running.
Open pluggable database: PDBORCL.
PDBORCL is already opened
********** CREATING PetStore DB USER **********************************************

User dropped.


User created.


Grant succeeded.

********** CREATING DB ENTRIES FOR PetStore Application ***************************

SQL*Plus: Release 12.1.0.2.0 Production on Tue Oct 11 05:55:45 2016

Copyright (c) 1982, 2014, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL>
Table created.


Table created.


Table created.


Table created.


Table created.


Table created.


Table created.


Table created.


Table created.


1 row created.

...

1 row created.


Commit complete.

SQL> Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
********** CREATING PETSTORE_DOMAIN (WEBLOGIC 10.3.6 - PETSTORE_DOMAIN) ***********
<< read template from "/u01/content/cloud-native-devops-workshop/app-2-cloud/template/petstore_domain_template.jar"
>>  succeed: read template from "/u01/content/cloud-native-devops-workshop/app-2-cloud/template/petstore_domain_template.jar"
<< find User "weblogic" as u1_CREATE_IF_NOT_EXIST
>>  succeed: find User "weblogic" as u1_CREATE_IF_NOT_EXIST
<< set u1_CREATE_IF_NOT_EXIST attribute Password to "********"
>>  succeed: set u1_CREATE_IF_NOT_EXIST attribute Password to "********"
<< write Domain to "/u01/wins/wls1036/user_projects/domains/petstore_domain"
...............................................................................................
>>  succeed: write Domain to "/u01/wins/wls1036/user_projects/domains/petstore_domain"
<< close template
>>  succeed: close template
********** STARTING ADMIN SERVER (WEBLOGIC 10.3.6 - PETSTORE_DOMAIN) **************
********** STARTING MSERVER1 SERVER (WEBLOGIC 10.3.6 - PETSTORE_DOMAIN) ***********
********** STARTING MSERVER2 SERVER (WEBLOGIC 10.3.6 - PETSTORE_DOMAIN) ***********
********** ADMIN SERVER (WEBLOGIC 10.3.6 - DOMAIN1036) HAS BEEN STARTED ***********
********** DEPLOY PETSTORE (WEBLOGIC 10.3.6 - PETSTORE_DOMAIN) ********************

Initializing WebLogic Scripting Tool (WLST) ...

Welcome to WebLogic Server Administration Scripting Shell

Type help() for help on available commands

************************ Create resources for PETSTORE application *****************************************
Connecting to t3://localhost:7001 with userid weblogic ...
Successfully connected to Admin Server 'AdminServer' that belongs to domain 'petstore_domain'.

Warning: An insecure protocol was used to connect to the
server. To ensure on-the-wire security, the SSL port or
Admin port should be used instead.

Location changed to edit tree. This is a writable tree with
DomainMBean as the root. To make changes you will need to start
an edit session via startEdit().

For more help, use help(edit)

Starting an edit session ...
Started edit session, please be sure to save and activate your
changes once you are done.
Saving all your changes ...
Saved all your changes successfully.
Activating all your changes, this may take a while ...
The edit lock associated with this edit session is released
once the activation is completed.
Activation completed
************************ Deploy PETSTORE application *****************************************
Deploying application from /u01/wins/wls1036/wlserver_10.3/common/deployable-libraries/jsf-2.0.war to targets petstore_cluster (upload=false) ...
<Oct 11, 2016 5:57:20 AM PDT> <Info> <J2EE Deployment SPI> <BEA-260121> <Initiating deploy operation for application, jsf [archive: /u01/wins/wls1036/wlserver_10.3/common/deployable-libraries/jsf-2.0.war], to petstore_cluster .>
.Completed the deployment of Application with status completed
Current Status of your Deployment:
Deployment command type: deploy
Deployment State       : completed
Deployment Message     : [Deployer:149194]Operation 'deploy' on application 'jsf [LibSpecVersion=2.0,LibImplVersion=1.0.0.0_2-0-2]' has succeeded on 'mserver2'
Deploying application from /u01/content/cloud-native-devops-workshop/app-2-cloud/petstore.12.war to targets petstore_cluster (upload=false) ...
<Oct 11, 2016 5:57:23 AM PDT> <Info> <J2EE Deployment SPI> <BEA-260121> <Initiating deploy operation for application, Petstore [archive: /u01/content/cloud-native-devops-workshop/app-2-cloud/petstore.12.war], to petstore_cluster .>
...Completed the deployment of Application with status completed
Current Status of your Deployment:
Deployment command type: deploy
Deployment State       : completed
Deployment Message     : [Deployer:149194]Operation 'deploy' on application 'Petstore' has succeeded on 'mserver2'
Starting application Petstore.
<Oct 11, 2016 5:57:33 AM PDT> <Info> <J2EE Deployment SPI> <BEA-260121> <Initiating start operation for application, Petstore [archive: null], to petstore_cluster .>
.Completed the start of Application with status completed
Current Status of your Deployment:
Deployment command type: start
Deployment State       : completed
Deployment Message     : [Deployer:149194]Operation 'start' on application 'Petstore' has succeeded on 'mserver2'
No stack trace available.
<Oct 11, 2016 5:57:36 AM PDT> <Warning> <JNDI> <BEA-050001> <WLContext.close() was called in a different thread than the one in which it was created.>
********** OPEN PETSTORE APPLICATION AT http://localhost:7003/petstore/faces/catalog.jsp
[oracle@localhost app-2-cloud]$
```

Petstore デモ・アプリケーションの確認のため、ブラウザを開きアプリケーションの URL を入力する:

- ***http://localhost:7003/petstore/faces/catalog.jsp***

![](images/on.prem.petstore.png)

virtualbox 環境では、2つの管理対象サーバに対するロードバランシング構成をとっていない。そのため、`7003` 番ポートと `7004` 番ポートが各管理対象サーバへの直接アクセスを行う。

#### AppToCloud ツールを用いたドメインのエクスポート

このチュートリアル用に用意していある VirtualBox 環境を使用していない場合、AppToCloud ツールを次のサイトからダウンローする必要がある:

- [AppToCloud ツールダウンロード](http://www.oracle.com/technetwork/topics/cloud/downloads/index.html#apptocloud)

また、PaaS のサービス・コンソール上のダウンロードセンターからもダウンロード可能である。コンソール上部の **ユーザ名** をクリックし、**Help** を選択し **Download Center** をクリックする。
このツールは、`a2c-zip-installer.zip` にアーカイブされており、適当な場所に展開して使用する。AppToCloud は移行対象のドメインの管理サーバが稼働しているマシンにインストールする必要がある。このチュートリアル用の VirtualBox 環境の場合は、次の場所に展開してある:

- `/u01/oracle_jcs_app2cloud`

該当のディレクトリに `a2c-healthcheck.sh` が存在している事を確認する。

```
[oracle@localhost app-2-cloud]$ ls -la /u01/oracle_jcs_app2cloud/bin
total 60
drwxr-xr-x. 2 oracle oracle 4096 Oct 10 09:01 .
drwxr-x---. 7 oracle oracle 4096 Oct 11 00:16 ..
-rw-r-----. 1 oracle oracle 7582 Aug  4 11:25 a2c-export.cmd
-rwxr-x---. 1 oracle oracle 7137 Aug  4 11:25 a2c-export.sh
-rw-r-----. 1 oracle oracle 8683 Aug  4 11:25 a2c-healthcheck.cmd
-rwxr-x---. 1 oracle oracle 8169 Aug  4 11:25 a2c-healthcheck.sh
-rw-r-----. 1 oracle oracle 6095 Aug  4 11:25 a2c-import.cmd
-rwxr-x---. 1 oracle oracle 5673 Aug  4 11:25 a2c-import.sh
```

AppToCloud ツールの `a2c-healthcheck` スクリプトは、オンプレミスの WebLogic Server ドメインとアプリケーションを Java Cloud Service に移動する準備として正常性を確認するために使用するヘルスチェック・ツールである。このスクリプトは次のパラメータを必要とする:

1. マシン上にインストールされている WebLogic Server の最上部のディレクトリ: この場所は、`ORACLE_HOME` として参照される。
2. 管理サーバの管理コンソールの URL
3. システム管理ユーザの認証情報
4. 実行結果の出力先

`a2c-healthcheck.sh` うを実行して、稼働中のドメインの確認とエクスポートを実施する。実行中に管理者パスワードを求められると、***welcome1*** を入力する

```
[oracle@localhost app-2-cloud]$ /u01/oracle_jcs_app2cloud/bin/a2c-healthcheck.sh -oh /u01/wins/wls1036 -adminUrl t3://localhost:7001 -adminUser weblogic -outputDir /u01/jcs_a2c_output
JDK version is 1.8.0_60-b27
A2C_HOME is /u01/oracle_jcs_app2cloud
/usr/java/latest/bin/java -Xmx512m -cp /u01/oracle_jcs_app2cloud/jcs_a2c/modules/features/jcsa2c_lib.jar -Djava.util.logging.config.class=oracle.jcs.lifecycle.util.JCSLifecycleLoggingConfig oracle.jcs.lifecycle.healthcheck.AppToCloudHealthCheck -oh /u01/wins/wls1036 -adminUrl t3://localhost:7001 -adminUser weblogic -outputDir /u01/jcs_a2c_output
The a2c-healthcheck program will write its log to /u01/oracle_jcs_app2cloud/logs/jcsa2c-healthcheck.log
Enter password:
Checking Domain Health
Connecting to domain

Connected to the domain petstore_domain

Checking Java Configuration
...
checking server runtime : mserver2
...
checking server runtime : mserver1
...
checking server runtime : AdminServer
Done Checking Java Configuration
Checking Servers Health

Done checking Servers Health
Checking Applications Health
Checking jsf#2.0@1.0.0.0_2-0-2
Checking Petstore
Done Checking Applications Health
Checking Datasource Health
Done Checking Datasource Health
Done Checking Domain Health

Activity Log for HEALTHCHECK

Informational Messages:

  1. JCSLCM-04037: Healthcheck Completed

An HTML version of this report can be found at /u01/jcs_a2c_output/reports/petstore_domain-healthcheck-activityreport.html

Output archive saved as /u01/jcs_a2c_output/petstore_domain.zip.  You can use this archive for the a2c-export tool.


a2c-healthcheck completed successfully (exit code = 0)
[oracle@localhost app-2-cloud]$
```

ヘルスチェック・ツールが正常終了する事を確認する (exit code: 0)。ヘルスチェックの出力結果のエラーメッセージ部分に問題があれば記述されるので対応した後、ヘルスチェック・ツールを再度実行する。
ヘルスチェック・ツールの結果は HTML ファイルのレポートとして確認できる。レポートファイルの名前と出力場所は、ツールの実行結果に表示される。

#### オンプレミス・データベースの Database Cloud Service への移行

Java Cloud Service インスタンスは、Oracle JRF スキーマをホストするために Database Cloud Service のデプロイが必須である。このスキーマは、Java Cloud Service のインスタンスを新規に作成した際に自動でプロビジョニングされる。

多くの Java アプリケーションはオンプレミスのデータベースを使用している。オラクルはアプリケーションが使用するデータベースも Database Cloud Service へ移行することを推奨している。AppToCloud を使用して Java Cloud Service インスタンスを作成すると、Database Cloud Service 上で稼働しているデータベースに対して既存の WebLogic Server に定義済みであるデータソースを関連付ける事が可能である。

このチュートリアルでは、Oracle JRF スキーマと Petstore デモ・アプリケーションのスキーマを同じ Database Cloud Service のデーターベースを使用する。

オンプレミスのデータベースを Database Cloud Service に移行するには、いくつかの手段がある。このチュートリアルでは、SQL スクリプトを使用し、デモ・アプリケーションに必要なスキーマ、テーブル、データを作成する。ここでは、次のパラメータともにスクリプトを実行する事で実施する:

- **Database Cloud Service 管理者名**: インスタンス作成時に変更なければ、***system***
- **Database Cloud Service パスワード**: インスタンス作成時に指定したパスワード
- **SSH 秘密鍵ファイル**: Database Cloud Service インスタンスに対応している秘密鍵を指定。[UI を用いた Database Cloud Service インスタンスの作成](,,/dbcs-create/README.md) によりインスタンスを作成している場合は、`/u01/content/cloud-native-devops-workshop` に配置されている ***privateKey*** を指定する。異なる場合は、パスとファイル名を指定する。
- **Database Cloud Service パブリック IP アドレス**: Database Cloud Service インスタンスの VM にアクセスするために使用するパブリック IP アドレスを指定
- **プラガブルデータベース名**: デフォルトでは **PDB1** が用いられる。インスタンス作成時に変更している場合は定義した名称を指定。

サービス・コンソールを使用してパブリック IP アドレスを確認するには、まず次の URL からサインインする。
- [https://cloud.oracle.com/sign-in](https://cloud.oracle.com/sign-in)

次に、ダッシュボード画面で Database Cloud Service のコンソールを開く。

![](images/open.dbcs.console.png)

Java Cloud Service に関連付けられ、Petstore デモ・アプリケーションのスキーマをホストしている Database Cloud Service インスタンスをクリックする。

![](images/open.dbcs.instance.details.png)

パブリック IP アドレスが確認できる。

![](images/dbcs.public.ip.png)

Petstore デモ・アプリケーション用のデータベース環境を準備するために、必須パラメータと共にスクリプトを実行する。実行結果は次のようになる:

	[oracle@localhost app-2-cloud]$ ./prepareDBCS.sh system <YOUR_DBCS_SYSTEM_PASSWORD ><YOUR_PRIVATEKEY_LOCATION> <YOUR_DBCS_PUBLIC_IP>
	Enter passphrase for key '../pk.openssh':
	create_user.sh                                                                                                     100%  301     0.3KB/s   00:00    
	create_user.sql                                                                                                    100%  101     0.1KB/s   00:00    
	petstore.sql                                                                                                       100%   55KB  55.3KB/s   00:00    
	Enter passphrase for key '../pk.openssh':

	SQL*Plus: Release 12.1.0.2.0 Production on Tue Oct 11 09:32:46 2016

	Copyright (c) 1982, 2014, Oracle.  All rights reserved.

	Last Successful login time: Mon Sep 12 2016 12:40:26 +00:00

	Connected to:
	Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production

	SQL> DROP USER petstore cascade
	        *
	ERROR at line 1:
	ORA-01918: user 'PETSTORE' does not exist


	SQL> SQL>
	User created.

	SQL>
	Grant succeeded.

	SQL> Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production

	SQL*Plus: Release 12.1.0.2.0 Production on Tue Oct 11 09:32:46 2016

	Copyright (c) 1982, 2014, Oracle.  All rights reserved.


	Connected to:
	Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production

	SQL> SQL>   2    3    4    5    6    7  
	Table created.

	...

	SQL>
	1 row created.

	SQL> SQL>
	Commit complete.

	SQL> Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
	[oracle@localhost app-2-cloud]$

このスクリプトは再実行可能となっている。実行失敗した場合は、パラメータを修正したりその他問題を対応して、再実行する。

Database Cloud Service は、`PETSTORE` スキーマに必須テーブルとデータを持っている。`PETSTORE` のパスワードは、Database Cloud Service の管理者パスワードが設定されている。

#### オンプレミス Welogic ドメインのエクスポート

エクスポート・ツールにより、ドメインの構成と Java アプリケーションのキャプチャーを行う。このツールにより、ヘルスチェック・ツールにより生成されたアーカイブ・ファイルを更新し、JSON ファイルを生成し、これらの AppToCloud ツールによる生成物を Storage Cloud Service の作成済みのストレージコンテナにアップロードする。

ここでは、Database Cloud Service インスタンスの作成時に提供されたストレージコンテナを使用する。ストレージコンテナを覚えていない場合は、Database Cloud Service の詳細ページで確認できる。

![](images/dbcs.storage.png)

Storage Cloud Service 上に新規にコンテナを作成したい場合は、以下のドキュメントに従って操作する:

- [Creating Containers](http://www.oracle.com/pls/topic/lookup?ctx=cloud&id=CSSTO00708)

AppToCloud ツールの `a2c-export.sh` スクリプトを実行することで、オンプレミス WebLogic Server ドメインとアプリケーションをキャプチャーし、Java Cloud Service 環境がアクセスできる ストレージコンテナに移動してくれる。このスクリプトは次のパラメータを要する:

1. WebLogic Server ドメインディレクトリの最上部
2. WebLogic Server のインストールディレクトリの最上部
3. クラウドストレージコンテナ名 (フォーマット: `Storage-<アイデンティティドメイン名>/<コンテナ名>`)
4. クラウドストレージコンテナ認証情報 (ユーザ名/パスワード)

上記のパラメータを使用して、`a2c-export.sh` を実行し、エクスポートとアップロードを実施する。以下、準備された virtualbox 環境で実施した実行例である:

	[oracle@localhost app-2-cloud]$ /u01/oracle_jcs_app2cloud/bin/a2c-export.sh -oh /u01/wins/wls1036 -domainDir /u01/wins/wls1036/user_projects/domains/petstore_domain -archiveFile /u01/jcs_a2c_output/petstore_domain.zip -cloudStorageContainer  <YOUR_CLOUD_CONTAINER_PATH> -cloudStorageUser <YOUR_CLOUD_STORAGE_USER>

```
[oracle@localhost jcs_a2c_output]$ /u01/oracle_jcs_app2cloud/bin/a2c-export.sh -oh /u01/wins/wls1036 -domainDir /u01/wins/wls1036/user_projects/domains/petstore_domain -archiveFile /u01/jcs_a2c_output/petstore_domain.zip -cloudStorageContainer Storage-<アイデンティティドメイン名>/Container_A2C -cloudStorageUser shinyay
JDK version is 1.8.0_60-b27
A2C_HOME is /u01/oracle_jcs_app2cloud
/usr/java/latest/bin/java -Xmx512m -DUseSunHttpHandler=true -cp /u01/oracle_jcs_app2cloud/jcs_a2c/modules/features/jcsa2c_lib.jar -Djava.util.logging.config.class=oracle.jcs.lifecycle.util.JCSLifecycleLoggingConfig oracle.jcs.lifecycle.discovery.AppToCloudExport -oh /u01/wins/wls1036 -domainDir /u01/wins/wls1036/user_projects/domains/petstore_domain -archiveFile /u01/jcs_a2c_output/petstore_domain.zip -cloudStorageContainer Storage-shinyay1612/Container_A2C -cloudStorageUser shinyay
The a2c-export program will write its log to /u01/oracle_jcs_app2cloud/logs/jcsa2c-export.log
Enter Storage Cloud password:
####<Jan 6, 2017 3:52:45 AM> <INFO> <AppToCloudExport> <getModel> <JCSLCM-02005> <Creating new model for domain /u01/wins/wls1036/user_projects/domains/petstore_domain>
####<Jan 6, 2017 3:52:45 AM> <INFO> <EnvironmentModelBuilder> <populateOrRefreshFromEnvironment> <FMWPLATFRM-08552> <Try to discover a WebLogic Domain in offline mode>
####<Jan 6, 2017 3:52:53 AM> <INFO> <EnvironmentModelBuilder> <populateOrRefreshFromEnvironment> <FMWPLATFRM-08550> <End of the Environment discovery>
####<Jan 6, 2017 3:52:53 AM> <WARNING> <ModelNotYetImplementedFeaturesScrubber> <transform> <JCSLCM-00579> <Export for Security configuration is not currently implemented and must be manually configured on the target domain.>
####<Jan 6, 2017 3:52:53 AM> <INFO> <AppToCloudExport> <archiveApplications> <JCSLCM-02003> <Adding application to the archive: Petstore from /home/oracle/cloud_native_workshop/app-2-cloud/petstore.12.war>
####<Jan 6, 2017 3:52:54 AM> <INFO> <AppToCloudExport> <archiveSharedLibraries> <JCSLCM-02003> <Adding library to the archive: jsf#2.0@1.0.0.0_2-0-2 from /u01/wins/wls1036/wlserver_10.3/common/deployable-libraries/jsf-2.0.war>
####<Jan 6, 2017 3:52:57 AM> <INFO> <AppToCloudExport> <run> <JCSLCM-02009> <Successfully exported model and artifacts to /u01/jcs_a2c_output/petstore_domain.zip. Overrides file written to /u01/jcs_a2c_output/petstore_domain.json>
####<Jan 6, 2017 3:52:57 AM> <INFO> <AppToCloudExport> <run> <JCSLCM-02028> <Uploading override file to cloud storage from /u01/jcs_a2c_output/petstore_domain.json>
####<Jan 6, 2017 3:53:01 AM> <INFO> <AppToCloudExport> <run> <JCSLCM-02028> <Uploading archive file to cloud storage from /u01/jcs_a2c_output/petstore_domain.zip>
####<Jan 6, 2017 3:53:17 AM> <INFO> <AppToCloudExport> <run> <JCSLCM-02009> <Successfully exported model and artifacts to https://<アイデンティティドメイン名>.storage.oraclecloud.com. Overrides file written to Storage-shinyay1612/Container_A2C/petstore_domain.json>

Activity Log for EXPORT

Informational Messages:

  1. JCSLCM-02030: Uploaded override file to Oracle Cloud Storage container Storage-<アイデンティティドメイン名>/Container_A2C
  2. JCSLCM-02030: Uploaded archive file to Oracle Cloud Storage container Storage-<アイデンティティドメイン名>/Container_A2C

Features Not Yet Implemented Messages:

  1. JCSLCM-00579: Export for Security configuration is not currently implemented and must be manually configured on the target domain.

An HTML version of this report can be found at /u01/jcs_a2c_output/reports/petstore_domain-export-activityreport.html

Successfully exported model and artifacts to https://<アイデンティティドメイン名>.storage.oraclecloud.com. Overrides file written to Storage-<アイデンティティドメイン名>/Container_A2C/petstore_domain.json


a2c-export completed successfully (exit code = 0)
[oracle@localhost jcs_a2c_output]$
```

エクスポート・ツールが正常終了した事を確認する (exit code: 0)。また生成された JSON ファイルの名前を確認する。エクスポート・ツールの実行結果のエラーメッセージ部分に問題があれば記述されるので対応した後、エクスポート・ツールを再度実行する。
エクスポート・ツールの結果は HTML ファイルのレポートとして確認できる。レポートファイルの名前と出力場所は、ツールの実行結果に表示される。

ドメインをエクスポートし、ファイルをストレージコンテナにアップロードすると、Java Cloud Service インスタンスの作成準備ができている。

#### AppToCloud を用いた Java Cloud Service のインスタンス作成

Java Cloud Service にソース・ドメインの構成とアプリケーションをインポートするには、AppToCloud ツールによって生成したファイルと新規に作成したインスタンスを関連付ける必要がある。

AppToCloud を用いたサービス・インスタンスの作成手順のほとんどは、標準のサービス・インスタンス作成手順と同じである。しかし、いくつか追加の手順がある:

- AppToCloud により生成した JSON ファイルの Storage Cloud 上の配置場所を指定する必要がある
- オンプレミスの WebLogic Server で定義していたデータソースと Database Cloud Serviceの関連付けを行う必要がある

次の URL から[サインイン](../common/sign.in.to.oracle.cloud.md)する。
- [https://cloud.oracle.com/sign-in](https://cloud.oracle.com/sign-in)

ダッシュボード画面で Java Cloud Service コンソールを開く。

![](images/10.dashboard.png)

初めてコンソール画面を開く場合、ウェルカム・ページが表示される。この場合は、**Go To Console** をクリックする。

![](images/11.jcs.welcome.png)

**Create Service** をクリックし、**Java Cloud Service — AppToCloud** を選択する。
![](images/12.jcs.console.png)

エクスポート・ツールで生成した JSON ファイルをアップロードした Storage Cloiud Service の情報を入力する。例:

- `Storage-<アイデンティティドメイン名>/<コンテナ名>/petsotre_domain.json`

また、クラウドストレージのユーザ名とパスワードを入力する:

![](images/13.app2cloud.storage.png)

**Service Level** と **Billing Frequency** を選択し、**Next** をクリックする。

![](images/14.service.billing.png)

**Software Release** を選択し、**Next** をクリックする。

![](images/15.sw.release.png)

**Software Edition** 選択肢、**Next** をクリックする。

![](images/16.sw.edition.png)

Java Cloud Service 詳細ページに以下の必須フィールドを入力する:

- Service name: **petstore**
- Description: オプション
- SSH public key: Database Cloud Service と同じキーを使用する場合は同じファイルまたは値を選択する。新規に作成する場合はダイアログが開かられる。
- Enable access to Administration Consoles: 有効
- Shape: デフォルト (OC3)
- Username: Weblogic 管理者ユーザ名
- Password: Weblogic 管理者パスワード
- Cluster size: JSON の構成に基づき、**2** が設定済み
- Cloud Storage container: ここでは、JSON ファイルが配置されていたパスを設定 例: `Storage-MyAccount/Container1`
- Cloud Storage Username: ストレージコンテナへのアクセスユーザ名
- Cloud Storage Password: ストレージコンテナへのアクセスパスワード
- Create Cloud Storage Container: このJava Cloud Service インスタンス 用に独立したコンテナを作成する場合に有効化
- Database: Database Cloud Service インスタンスの指定。Petstore デモ・アプリケーション用に準備したインスタンスを推奨
- Database Administrator Username: **sys**
- Database Password: データベース管理者のパスワード
- PDB Name: ***デフォルト***: PDB1
- Provision Load Balancer: **Yes**
- Load Balancer details: **デフォルト**

![](images/17.jcs.details.png)

**Additional Service Details** 画面では、**Application Data Source** で ***PETSTORE*** を選択する。そして以下のパラメータを設定する:

- DBCS Instance: Petstore アプリケーション用に準備したインスタンスを選択
- Username: **petstore** を指定。データソースにこのユーザ名で接続する。
- Password: データベース管理者のパスワードを指定。(先に実施したスクリプトでは **petstore** と同じパスワードを設定)

**OK** をクリックし、変更内容を受諾する。**Next** をクリックする。

![](images/18.jcs.datasource.png)

確認ページが表示される。選択内容に相違なければ、**Create** をクリックする。

![](images/19.jcs.summary.png)


**Activity** タブで進捗状況とステータスをモニターできる。サービス・インスタンスがプロビジョニングされて稼働したら、AppToCloud ツールの生成ファイルをサービス・インスタンスにインポートする準備ができている。

#### サービス・インスタンスへのアプリケーションのインポート

AppToCloud によるJava Cloud Service インスタンスの作成が終了したら、オンプレミス環境から収集したアプリケーションとその他のドメインリソースをインポートし、サービスインスタンスに自動で反映を行う。

AppToCloud で作成したサービス・インスタンスを表示し、サービス・インスタンス名の隣にあるハンバーガーメニューから **AppToCloud Import** を選択する。

![](images/21.jcs.ready.png)

確認が表示されたら、**Yes** をクリックする。

![](images/22.confirm.app.import.png)

**Activity** タブでインポートの進捗状況を確認する事ができる。正常にインポートが終了すると、オンプレミス環境のソースドメインで使用していたアプリケーション及びその他のドメインリソースがサービス・インスタンスに反映されている事が確認できる。

Java Cloud Service 上で Petstore デモ・アプリケーションを確認するには、ロードバランサのパブリック IP アドレスを確認する必要がある。Java Cloud Service インスタンスの **petstore** をクリックする。

![](images/22.petstore.instance.png)

ロードバランサのパブリック IP アドレスを確認する

![](images/23.lb.ip.png)

ブラウザを開き、ロードバランサのパブリック IP アドレスを入力し、続けて `/petstore/faces/catalog.jsp` を入力し、アプリケーションにアクセスする。
アクセスURL例:
- `http://140.86.0.54/petstore/faces/catalog.jsp`

ロードバランサの IP アドレスを忘れないようにする。

![](images/24.petstore.jcs.png)

以上で、オンプレミス環境の Java EE 5 アプリケーションを Java Cloud Service 上に AppToCloud を用いて移行する事ができた。

詳細な **AppToCloud 移行** に関する情報は、[documentation](http://docs.oracle.com/cloud/latest/jcs_gs/JSCUG/GUID-C1F6804C-8D1C-457C-AC4A-28DD85691D09.htm#JSCUG-GUID-C1F6804C-8D1C-457C-AC4A-28DD85691D09) を参照。
