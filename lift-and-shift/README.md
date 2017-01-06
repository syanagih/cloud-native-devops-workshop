![](../common/images/customer.logo.png)
---
# ORACLE Cloud-Native DevOps workshop #
----
## WebLogic Server 12cR2 から Java Cloud Service へドメイン・パーティションの移動

### 説明

ドメイン・パーティションをエクスポート＆インポート操作により WebLogic 管理者は、あるドメインから別のドメインへ簡単に移動できるようになる。また、ドメイン・パーティションにはデプロイしたアプリケーションも含まれて移動できる。この機能は、ドメイン間でのパーティションの複製や、開発環境からテスト環境、そして本番環境へのドメインの移動に役立つものである。

ドメイン・パーティションのエクスポートにより、パーティションのバックアップが作成され、アーカイブ・ファイルに格納される。この機能により、ソース・ドメインからドメイン・パーティションの全ての構成とデータをエクスポートする事が可能である。少しの構成変更により、パーティション・アーカイブを別の WebLogic Server マルチテナント インスタンスにインポートする事ができる。ターゲット環境やセキュリティ・レルムのようなドメインに依存する情報の更新が必要になる場合がある。また、その他にパーティション構成の属性を更新して有効にする必要がある場合がある。
パーティションがソース・ドメインからエクスポートされると、パーティション・アーカイブにパッケージされる。パーティション・アーカイブには、次のものが含まれる:

- パーティション構成
- パーティションに含まれるリソース・グループ
- リソース・グループに参照されるリソース・グループ・テンプレート
- ファイルシステムのコンテンツ: <パーティション・ファイルシステム>/config ディレクトリ
- オプション: パーティションにデプロイされているアプリケーション・バイナリ及び構成

アプリケーションの実行状態や、アプリケーション固有のランタイム構成はパーティション・アーカイブには含まれない。例えば、キュー内部の JMS メッセージや、組み込み LDAP レルム内のユーザ情報はエクスポートされない。

このチュートリアルでは、次の操作を行う:

- オンプレミス環境の開発モードの `vanilla domain` からドメイン・パーティションのエクスポート
- Java Cloud Service 上の本番モードで稼働しているドメインにドメイン・パーティションをインポート

### チュートリアルについて

このチュートリアルでは、エクスポート＆インポート機能によりオンプレミス環境の WebLogic Server で稼働しているドメイン・パーティションを Java Cloud Service 上の WebLogic Server に移動を行う

![](images/part2.generic.overview.png)

データベースの移行はこのチュートリアルのスコープ外とする。これによって、全ての DDL と DML を Database Cloud Service 環境にコピーして実行するスクリプトを提供する。また、このスクリプトは WebLogic 12.2.1 からエクスポートしたパーティション・アーカイブを Java Cloud Service 上でインポートするためにコピーする。

### 前提

- 以下のチュートリアルを実施済みである事
  - [DPCT (Domain to Partition Conversion Tool) を用いた WebLogic 11g ドメインから WebLogic 12cR2 ドメイン・パーティションへの変換](../dpct/README.md)
- Provided VirtualBox image or [custom environment prepared](../common/vbox.vm.md) for this tutorial
- [UI を用いた Java Cloud Service インスタンスの作成](../jcs-create/README.md) 又は、Java Cloud Service 上に構成した WebLogic Server 12cR2 環境が稼働している事
- [UI を用いた Database Cloud Service インスタンスの作成](../dbcs-create/README.md) 又は、ターゲット環境のJava Cloud Service に対して構成されている Database Cloud Service が稼働している事

### 手順

#### オンプレミス Domain1221 からドメイン・パーティションのエクスポート

オンプレミス環境の WebLogic Server 12cR2 の管理コンソールをブラウザで開きログインする。

- ユーザ名: **weblogic**
- パスワード: **welcome1**

そして左ペインから **Domain Partitions** を選択し、**Microcontainer1** のチェックボックスを選択して **Export** をクリックする。

![](images/console.12cR2.domain.partition.export.png)


Check the box for **Include Application Bits** のチェックボックスを選択し、パスに `<クローンしたGitリポジトリ>/lift-and-shift/JCS` を設定して **OK** をクリックする。

![](images/console.12cR2.domain.partition.export.details.png)

ターミナルを開き、`<クローンしたGitリポジトリ>/lift-and-shift` へ移動する。`ls -la` コマンドで ***Microcontainer1.zip*** と ***Microcontainer1-attributes.json*** が作成されている事を確認する。これらのファイルは、Java Cloud Service にパーティションをインポートする際に必要となる。

```
[oracle@localhost lift-and-shift]$ ls -la JCS
total 7380
drwxrwx---. 2 oracle oracle    4096 Jan  5 03:31 .
drwxrwx---. 4 oracle oracle      71 Jan  5 03:26 ..
-rwxrwx---. 1 oracle oracle     716 Dec  4 19:48 createVT.py.sample
-rwxrwx---. 1 oracle oracle      90 Dec  4 19:48 createVT.sh
-rw-r-----. 1 oracle oracle    1077 Jan  5 03:31 Microcontainer1-attributes.json
-rw-r-----. 1 oracle oracle 7538419 Jan  5 03:31 Microcontainer1.zip
```


クラウド・サービスを準備するために `prepareCloudServices.sh` を実行する:

```
$ [oracle@localhost lift-and-shift]$ ./prepareCloudServices.sh
```
(このスクリプトは、まず DBCS に必要なファイルをコピーし、サンプルデータをデータベースに生成する。ここでは SQL スクリプトを使用している。一般的には、ローカルデータベースをアンプラグし、DBCS インスタンスにコピーしてプラガブル・データベースをプラグインする事ができる。または、データベースのエクスポート＆インポートが行える。しかし、ここではシンプルにするために SQL スクリプトを使用している。パーティションをエクスポートしたアーカイブファイルとJSONファイルを JCS インスタンス の /tmp にコピーし、パーミションを設定する。またインポートしたパーティションが使用する仮想ターゲット `Microcontainer1-AdminServer-virtualTarget` を WLST で作成を行う。)

	../cloud-utils/environment.properties found.
	Identity domain: paasdemo16
	Java Cloud Service Public IP address: 129.144.18.102
	Database Cloud Service Public IP address: 129.144.18.253
	[INFO] Scanning for projects...
	[INFO]                                                                         
	[INFO] ------------------------------------------------------------------------
	[INFO] Building LiftAndShift 1.0.0-SNAPSHOT
	[INFO] ------------------------------------------------------------------------
	Downloading: https://repo.maven.apache.org/maven2/org/codehaus/mojo/properties-maven-plugin/1.0-alpha-2/properties-maven-plugin-1.0-alpha-2.pom
	Downloaded: https://repo.maven.apache.org/maven2/org/codehaus/mojo/properties-maven-plugin/1.0-alpha-2/properties-maven-plugin-1.0-alpha-2.pom (4 KB at 4.5 KB/sec)
	Downloading: https://repo.maven.apache.org/maven2/org/codehaus/mojo/mojo-parent/21/mojo-parent-21.pom
	...
	...
	...             
	Downloaded: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/1.5.6/plexus-utils-1.5.6.jar (245 KB at 1229.5 KB/sec)
	[INFO]
	[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ LiftAndShift ---
	[INFO] Using 'UTF-8' encoding to copy filtered resources.
	[INFO] skip non existing resourceDirectory /u01/content/cloud-native-devops-workshop/lift-and-shift/src/main/resources
	[INFO]
	[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ LiftAndShift ---
	[INFO] No sources to compile
	[INFO]
	[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ LiftAndShift ---
	[INFO] Using 'UTF-8' encoding to copy filtered resources.
	[INFO] skip non existing resourceDirectory /u01/content/cloud-native-devops-workshop/lift-and-shift/src/test/resources
	[INFO]
	[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ LiftAndShift ---
	[INFO] No sources to compile
	[INFO]
	[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ LiftAndShift ---
	[INFO] Tests are skipped.
	[INFO]
	[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ LiftAndShift ---
	[WARNING] JAR will be empty - no content was marked for inclusion!
	[INFO] Building jar: /u01/content/cloud-native-devops-workshop/lift-and-shift/target/LiftAndShift.jar
	[INFO]
	[INFO] --- maven-install-plugin:2.4:install (default-install) @ LiftAndShift ---
	[INFO] Installing /u01/content/cloud-native-devops-workshop/lift-and-shift/target/LiftAndShift.jar to /home/oracle/.m2/repository/com/oracle/wins/cloud/LiftAndShift/1.0.0-SNAPSHOT/LiftAndShift-1.0.0-SNAPSHOT.jar
	[INFO] Installing /u01/content/cloud-native-devops-workshop/lift-and-shift/pom.xml to /home/oracle/.m2/repository/com/oracle/wins/cloud/LiftAndShift/1.0.0-SNAPSHOT/LiftAndShift-1.0.0-SNAPSHOT.pom
	[INFO]
	[INFO] --- maven-antrun-plugin:1.8:run (replaceJCS) @ LiftAndShift ---
	Downloading: https://repo.maven.apache.org/maven2/commons-net/commons-net/1.4.1/commons-net-1.4.1.pom
	Downloaded: https://repo.maven.apache.org/maven2/commons-net/commons-net/1.4.1/commons-net-1.4.1.pom (5 KB at 46.4 KB/sec)
	...
	...
	...
	Downloaded: https://repo.maven.apache.org/maven2/ant/ant-commons-net/1.6.5/ant-commons-net-1.6.5.jar (35 KB at 134.1 KB/sec)
	[INFO] Executing tasks

	main:
	     [copy] Copying 2 files to /u01/content/cloud-native-devops-workshop/lift-and-shift/JCS
	[INFO] Executed tasks
	[INFO]
	[INFO] --- maven-antrun-plugin:1.8:run (replaceDBCS) @ LiftAndShift ---
	[INFO] Executing tasks

	main:
	     [copy] Copying 2 files to /u01/content/cloud-native-devops-workshop/lift-and-shift/DBCS
	[INFO] Executed tasks
	[INFO]
	[INFO] --- maven-antrun-plugin:1.8:run (copy2DBCS) @ LiftAndShift ---
	[INFO] Executing tasks

	main:
	     [echo] Copy artifacts to DBCS.
	      [scp] Connecting to 129.144.18.253:22
	      [scp] done.
	  [sshexec] Connecting to 129.144.18.253:22
	  [sshexec] cmd : chmod 755 /tmp/petstore.sql /tmp/prepareDBCS.sh
	  [sshexec] Connecting to 129.144.18.253:22
	  [sshexec] cmd : /tmp/prepareDBCS.sh

	SQL*Plus: Release 12.1.0.2.0 Production on Thu Sep 15 13:40:48 2016
	Copyright (c) 1982, 2014, Oracle.  All rights reserved.

	Last Successful login time: Wed Sep 14 2016 23:00:21 +00:00

	Connected to:
	Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production

	SQL>  drop user petstore cascade
	           *
	ERROR at line 1:
	ORA-01918: user 'PETSTORE' does not exist

	SQL> SQL>

	User created.

	SQL>

	Grant succeeded.

	SQL> Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production

	SQL*Plus: Release 12.1.0.2.0 Production on Thu Sep 15 13:40:49 2016
	Copyright (c) 1982, 2014, Oracle.  All rights reserved.

	Connected to:
	Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production

	SQL>

	Table created.
	...
	...
	...
	1 row created.

	Commit complete.

	SQL> Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production

	[INFO] Executed tasks
	[INFO]
	[INFO] --- maven-antrun-plugin:1.8:run (copyScript2JCS) @ LiftAndShift ---
	[INFO] Executing tasks

	main:
	     [echo] Copy artifacts to JCS.
	      [scp] Connecting to 129.144.18.102:22
	      [scp] done.
	  [sshexec] Connecting to 129.144.18.102:22
	  [sshexec] cmd : chmod 755 /tmp/Microcontainer1-attributes.json /tmp/Microcontainer1.zip /tmp/createVT.sh /tmp/createVT.py
	  [sshexec] Connecting to 129.144.18.102:22
	  [sshexec] cmd : sudo su - oracle -c /tmp/createVT.sh oracle

	Initializing WebLogic Scripting Tool (WLST) ...

	Welcome to WebLogic Server Administration Scripting Shell

	Type help() for help on available commands

	Cluster name: winsdemoWLS_cluster
	Connecting to t3://winsdemoWLS-wls-1.compute-paasdemo16.oraclecloud.internal:7001 with userid weblogic ...
	Successfully connected to Admin Server "winsdemo_adminserver" that belongs to domain "winsdemoWLS_domain".

	Warning: An insecure protocol was used to connect to the server.
	To ensure on-the-wire security, the SSL port or Admin port should be used instead.

	Location changed to edit tree. 	 
	This is a writable tree with DomainMBean as the root. 	 
	To make changes you will need to start an edit session via startEdit().
	For more help, use help('edit').

	************************ Creating Virtual Target Microcontainer1-AdminServer-virtualTarget ****************************
	Starting an edit session ...
	Started edit session, be sure to save and activate your changes once you are done.
	Activating all your changes, this may take a while ...
	The edit lock associated with this edit session is released once the activation is completed.
	Activation completed
	Disconnected from weblogic server: winsdemo_adminserver
	[INFO] Executed tasks
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 01:03 min
	[INFO] Finished at: 2016-09-15T06:41:35-07:00
	[INFO] Final Memory: 21M/491M
	[INFO] ------------------------------------------------------------------------
	Database Cloud Service has been prepared.
	Java Cloud Service has been prepared.

#### Java Cloud Service へドメイン・パーティションをインポート

ブラウザを開き、Java Cloud Service の詳細ページを開く。ハンバーガー・メニューをクリックし、**Open WebLogic Server Console** を選択し、インポート後に必要な修正を管理コンソールで完了させる。この変更作業は、WLST によって自動化する事も可能である。
 The goal is here to demonstrate the Java Cloud Service capabilities.

![](images/jcs.console.open.png)

新しいタブが開かれると、証明書に関する警告が表示される。これは、商用 SSL 証明書をインストールしていないためで、デフォルトの状態ではデモ版を使用している。アクセスするために例外サイトとして追加を行う。何か問題が他に生じたらホストマシンのブラウザを使用する事も可能である。

![](images/jcs.console.untrusted.png)

![](images/jcs.console.untrusted.exception.png)

WebLogic ログイン情報を入力し、**Login** をクリックする。

![](images/jcs.console.login.png)

**Domain Partitions** をクリックし、**Lock & Edit** をクリックする。

![](images/jcs.console.domain.partitions.png)

**Import** をクリックし、***Path*** に `/tmp/Microcontainer1.zip` を入力し、**OK** をクリックする。

![](images/jcs.console.domain.partition.import.png)

**Lock & Edit** をクリックし、**Services** -> **Data Sources** -> **PetstoreDB** を選択する。

![](images/jcs.console.lock.and.edit.png)

**Connection Pool** タブをクリックし、***PetstoreDB*** データソースの **URL** を修正する:

変更前
- *jdbc:oracle:thin:@localhost:1521/pdborcl*

変更後
- **jdbc:oracle:thin:@<DBCS_INSTANCE_NAME>:1521/PDB1.<DOMAIN_ID>.oraclecloud.internal**

- DBCS_INSTANCE_NAME: DBCS サービスインスタンス名
- DOMAIN_ID: アイデンティティドメイン名

変更したら、**Save** をクリックする。

![](images/jcs.jdbc.pool.url.change.png)

**Activate Changes** をクリックする。

![](images/jcs.console.activate.changes.png)

左ペインから **Domain Partitions** を選択し、 **Control** タブをクリックする。***Microcontainer1*** のチェックボックスを選択し、**Start** をクリックする。

![](images/jcs.console.domain.partition.select.png)

確認画面で **Yes** をクリックする。

![](images/jcs.console.domain.partition.start.png)

再読み込みボタンをクリックする。ドメイン・パーティションが **RUNNING** 状態になったら、ブラウザからアプリケーションにアクセスする。

- [http://<JCSインスタンスのパブリックIP>/petstore/faces/catalog.jsp]()

![](images/petstore.on.jcs.png)


### Summary ###

ドメイン・パーティションをエクスポート＆インポート操作により WebLogic 管理者は、あるドメインから別のドメインへ簡単に移動できるようになる。また、ドメイン・パーティションにはデプロイしたアプリケーションも含まれて移動できる。この機能は、ドメイン間でのパーティションの複製や、開発環境からテスト環境、そして本番環境へのドメインの移動に役立つものである。
