![](../common/images/customer.logo.png)
---
# ORACLE Cloud-Native DevOps workshop #
----
## DPCT (Domain to Partition Conversion Tool) を用いた WebLogic 11g ドメインから WebLogic 12cR2 ドメイン・パーティションへの変換

### 説明


**Domain to Partition Conversion Tool** (**DPCT**) は、ソースドメインを調べ、リソースやデプロイされているアプリケーション、その他設定を含んだアーカイブを生成するユーティリティ・ツールである。これにより、ソースドメインに相当する新しいドメイン・パーティションを WebLogic Server 12.2.1 上にインポートする事ができる。JSON フォーマットで生成されているオーバーライド用ファイルを編集する事で、アーカイブを作成した際に関連するアーティファクトに使われてりるターゲットや名前を編集する事ができる。
DPCT は、WebLogic Server ***10.3.6***、***12.1.1***、***12.1.2***、***12.1.3*** のドメインをソースとして使用する事をサポートしており、WebLogic Server 12.2.1 ドメイン・パーティションに簡単な操作で変換する事ができる。

### チュートリアルについて

このチュートリアルは、既存の WebLogic Server 11g ドメインを WebLogic Server 12cR2 ドメイン・パーティションにインポートできるフォーマットに変換し、変換したドメインを WebLogic Serbver 12cR2 にインポートを実施する。

![](images/part1.generic.overview.png)

使用する VirtualBox VM には、既に WebLogic Server 10.3.6 のドメイン `Domain1036` を次の場所に作成している:

- `/u01/wins/wls1036/user_projects/domains/Domain1036`

このドメインは `管理サーバ` のみの構成であり、データソースとして `PetstoreDB` を作成している。また、共有ライブラリとして `JSF-2.0.war` をデプロイし、アプリケーションとして `petstore.12.war` が管理サーバ上にデプロイされている。

これが **ソース・ドメイン** となり、DPCT を用いてドメイン・パーティション・イメージに変換を行うものである。変換すると、`Domain1036.zip` と `Domain1036-attributes.json` が生成される。

また、以下の場所に WebLogic Server 12cR2 のドメインも作成済みである:

- `/u01/wins/wls1221/user_projects/domains/Domain1221`

これは未構成の空のドメインであり、管理サーバのみを含んでいる状態である。これが、**ターゲット・ドメイン** であり、ドメイン・パーティションとして、`Domain1036` をインポートするドメインである。

### 前提

- サンプル・アプリケーションをデプロイしている WebLogic Server 11g ドメインが構成済みである事
- WebLogic Server 12cR2 ドメインが構成済みである事
- Oracle Database がサンプル・アプリケーションのデータを格納している事

### 手順

#### 各ドメインの管理サーバの起動

ターミナル・ウインドウを開き、`<クローンしたGitリポジトリ>/dpct` に移動する。

```
$ [oracle@localhost Desktop]$ cd /u01/content/cloud-native-devops-workshop/dpct
```

事前に準備してあるスクリプトを使用して環境設定を行う。
まず、CDB と PDB (PDBORCL) を起動する。そして、**petstore** ユーザを *pdborcl* データベースに作成し、サンプルデータを作成する。
次に、以下の環境変数を設定し、

- **JAVA_HOME**: **/usr/java/jdk1.7.0_79/**
- **MW_HOME**: **/u01/wins/wls1036/**

***Domain1036*** の管理サーバを起動する。

最後に、以下の環境変数を設定し、

- **JAVA_HOME**: **/usr/java/latest/** (JDK 8)
- **MW_HOME**: **/u01/wins/wls1221/**

***Domain1221*** の管理サーバを起動する。

```
[oracle@localhost dpct]$ ./prepareLocalEnv.sh system welcome1
Oracle database (sid: orcl) is running.
Open pluggable database: PDBORCL.
PDBORCL is already opened
********** CREATING PetStore DB USER **********************************************
 drop user petstore cascade
           *
ERROR at line 1:
ORA-01918: user 'PETSTORE' does not exist



User created.


Grant succeeded.

********** CREATING DB ENTRIES FOR PetStore Application ***************************

SQL*Plus: Release 12.1.0.2.0 Production on Wed Jan 4 19:52:22 2017

Copyright (c) 1982, 2014, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL>
Table created.
...
...
...
1 row created.


Commit complete.

SQL> Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
********** STARTING ADMIN (WEBLOGIC 10.3.6 - DOMAIN1036) **************************
********** ADMIN SERVER (WEBLOGIC 10.3.6 - DOMAIN1036) HAS BEEN STARTED ***********
********** STARTING ADMIN SERVER (WEBLOGIC 12.2.1 - DOMAIN1221) *******************
********** ADMIN SERVER (WEBLOGIC 12.2.1 - DOMAIN1221) HAS BEEN STARTED ***********
```

ブラウザに戻り、アプリケーションの URL にアクセスする:

- [http://localhost:6001/petstore/faces/catalog.jsp](http://localhost:6001/petstore/faces/catalog.jsp)

または、次のブックマークを使用してもよい:

![](images/call.petstore.on.11g.png)

この **PetStore** は、Sun のエンジニアが 2009 年に　Java EE 5 及びその他フィーチャのデモンストレーションのために作成したアプリケーションである。猫や、犬、鳥といった動物をクリックする事ができ、また右上部のメニューからアプリケーションの実行確認もできる。ただし、外部サービスの API 変更により動作しない機能もある。

![](images/petstore.on.11g.png)


#### Domain to Partition Conversion Tool の実行

Domain to Partition Conversion Tool は次の場所に配置済みである:

- `/u01/dpct`

ターミナル・ウインドウに戻り、環境変数 **JAVA_HOME** が適切に設定されているか確認する。
Go back to terminal window and verify if **JAVA_HOME** variable is properly set.

```
[oracle@localhost dpct]$ echo $JAVA_HOME
/usr/java/latest
[oracle@localhost dpct]$ java -version
java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
```

`java version "1.8.0_60"` と表示されていれば、DPCT を実行要件を満たたしている。

DPCT の配置ディレクトリに移動し、DPCT を実行する。

```
$ [oracle@localhost dpct]$ cd /u01/dpct
$ [oracle@localhost dpct]$ ./exportDomainForPartition.sh /u01/wins/wls1036/ /u01/wins/wls1036/user_projects/domains/Domain1036/
```

このコマンドを実行すると、*Domain1036.zip* と *Domain1036-attributes.json* が 以下の場所に生成される:

- `/u01/dpct/outDir/folder`

```
CLASSPATH=/u01/dpct/com.oracle.weblogic.management.tools.migration.jar::/u01/wins/wls1036/patch_wls1036/profiles/default/sys_manifest_classpath/weblogic_patch.jar:/u01/wins/wls1036/patch_ocp371/profiles/default/sys_manifest_classpath/weblogic_patch.jar:/usr/java/latest/lib/tools.jar:/u01/wins/wls1036/wlserver_10.3/server/lib/weblogic_sp.jar:/u01/wins/wls1036/wlserver_10.3/server/lib/weblogic.jar:/u01/wins/wls1036/modules/features/weblogic.server.modules_10.3.6.0.jar:/u01/wins/wls1036/wlserver_10.3/server/lib/webservices.jar:/u01/wins/wls1036/modules/org.apache.ant_1.7.1/lib/ant-all.jar:/u01/wins/wls1036/modules/net.sf.antcontrib_1.1.0.0_1-0b2/lib/ant-contrib.jar::/u01/wins/wls1036/utils/config/10.3/config-launch.jar::/u01/wins/wls1036/wlserver_10.3/common/derby/lib/derbynet.jar:/u01/wins/wls1036/wlserver_10.3/common/derby/lib/derbyclient.jar:/u01/wins/wls1036/wlserver_10.3/common/derby/lib/derbytools.jar::
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0

Initializing WebLogic Scripting Tool (WLST) ...

Welcome to WebLogic Server Administration Scripting Shell

Type help() for help on available commands

WLST homepath:
/u01/dpct/tmpwlst
['/u01/wins/wls1036/wlserver_10.3/server/lib/weblogic.jar/Lib', '__classpath__', '/u01/wins/wls1036/wlserver_10.3/server/lib/weblogic.jar', '/u01/wins/wls1036/wlserver_10.3/common/wlst/modules/jython-modules.jar/Lib', '/u01/wins/wls1036/wlserver_10.3/common/wlst', '/u01/wins/wls1036/wlserver_10.3/common/wlst/lib', '/u01/wins/wls1036/wlserver_10.3/common/wlst/modules', '/u01/dpct/tmpwlst', '/u01/dpct/tmpwlst/lib', '/u01/dpct/tmpwlst/modules', '.']
Export a non-MT domain for migration
from domain path: /u01/wins/wls1036/user_projects/domains/Domain1036/
reading domain ......
domain name: Domain1036
drw-   AppDeployment
drw-   EmbeddedLDAP
drw-   JDBCSystemResource
drw-   Library
drw-   Security
drw-   SecurityConfiguration
drw-   Server
working on Server, listing ...
drw-   AdminServer
writing server for AdminServer
working on AppDeployment, listing ...
drw-   petstore
writing app-deployment for petstore
-rw-   AltDescriptorPath                             null
-rw-   AltWLSDescriptorPath                          null
-rw-   ApplicationIdentifier                         null
-rw-   ApplicationName                               null
-rw-   CompatibilityName                             null
-rw-   DeploymentOrder                               100
-rw-   DeploymentPrincipalName                       null
-rw-   InstallDir                                    null
-rw-   ModuleType                                    war
-rw-   Name                                          petstore
-rw-   Notes                                         null
-rw-   PlanDir                                       null
-rw-   PlanPath                                      null
-rw-   SecurityDdModel                               DDOnly
-rw-   SourcePath                                    /u01/wins/wls1036/user_projects/applications/Domain1036/petstore.12.war
-rw-   StagingMode                                   null
-rw-   Target                                        AdminServer
-rw-   ValidateDdSecurityData                        false
-rw-   VersionIdentifier                             null
working on Library, listing ...
drw-   jsf#2.0@1.0.0.0_2-0-2
writing library for jsf#2.0@1.0.0.0_2-0-2
-rw-   AltDescriptorPath                             null
-rw-   AltWLSDescriptorPath                          null
-rw-   ApplicationIdentifier                         null
-rw-   ApplicationName                               null
-rw-   CompatibilityName                             null
-rw-   DeploymentOrder                               100
-rw-   DeploymentPrincipalName                       null
-rw-   InstallDir                                    null
-rw-   ModuleType                                    war
-rw-   Name                                          jsf#2.0@1.0.0.0_2-0-2
-rw-   Notes                                         null
-rw-   PlanDir                                       null
-rw-   PlanPath                                      null
-rw-   SecurityDdModel                               DDOnly
-rw-   SourcePath                                    /u01/wins/wls1036/wlserver_10.3/common/deployable-libraries/jsf-2.0.war
-rw-   StagingMode                                   null
-rw-   Target                                        AdminServer
-rw-   ValidateDdSecurityData                        false
-rw-   VersionIdentifier                             null
working on JDBCSystemResource, listing ...
drw-   PetstoreDB
writing jdbc-system-resource for PetstoreDB
processing component-specific export handlers
create archive zip file ...
domain migration archive file: /u01/dpct/outDir/Domain1036.zip
generating domain migration archive completed.


Exiting WebLogic Scripting Tool.

Export Weblogic domain completed to outDir.
total 9932
-rw-r-----. 1 oracle oracle      823 Jan  4 21:37 Domain1036-attributes.json
-rw-r-----. 1 oracle oracle 10164844 Jan  4 21:37 Domain1036.zip
```

#### WebLogic 12.2.1 マルチテナント環境へのドメイン・パーティションのインポート

ブラウザに戻り、`Domain1221` の管理コンソール URL にアクセスする:

- [http://localhost:9001/console](http://localhost:9001/console)

または、次のブックマークを使用してもよい。

![](images/call.console.12cR2.png)

ユーザ名とパスワードは以下の通りに入力し、**Login** をクリックする:

- ユーザ名: **weblogic**
- パスワード: **welcome1**

![](images/console.12cR2.login.png)

左ペインの Domain Structure で、**Domain Partitions** をクリックする。

![](images/console.12cR2.domain.partitions.png)

**Import** をクリックし、, ドメイン・パーティション名に ***Microcontainer1*** と設定し、パスに **/u01/dpct/outDir/Domain1036.zip** を入力して `OK` をクリックする。

![](images/console.12cR2.import.png)

**Environment** -> **Virtual Targets** をクリックし、**Microcontainer1-AdminServer-virtualTarget** をクリックする。

![](images/console.12cR2.virtual.targets.png)

Modify the URI プレフィックスを "/Microcontainer1" から "/" に変更し、ポート・オフセットに ***100***　を設定し **Save** をクリックする。

![](images/console.12cR2.virtual.targets.modify.png)


 **Deployments** を選択する。、, as `jsf-2.0.war` は JSF 2.0 の実装であり、Java EE 6 の部品として WebLogic Server 12.2.1 にバンドルされている。よって、Java EE 6 以前のコンテナであった WebLogic Server 11g 環境では必要とされたが不要となる。`jsf(2.0,1.0.0.0_2-0-2)` を選択し、**Delete** をクリックする。

![](images/console.12cR2.remove.deployments.png)

**Domain Partitions** をクリックし、**Control** タブをクリックする。`Microcontainer1` のチェックボックスを選択し、**Start** をクリックしてドメイン・パーティションを起動する。

![](images/console.12cR2.domain.partition.start.png)

5秒程度してから、**Domain Partitions** の **Configuration** タブをクリックする。`Microcontainer1` パーティションの状態が ***RUNNING*** になっている。

![](images/console.12cR2.domain.partition.running.png)

`Petstore` デモ・アプリケーションにアクセスする:
- [http://localhost:9101/petstore/faces/catalog.jsp](http://localhost:9101/petstore/faces/catalog.jsp)

または、次のブックマークを使用してもよい。

![](images/call.petstore.on.12cR2.png)

![](images/petstore.on.12cR2.png)
