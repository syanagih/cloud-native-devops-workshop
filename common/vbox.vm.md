![](../common/images/customer.logo.png)
---
# ORACLE Cloud-Native DevOps workshop
-----
## オンプレミス環境の準備

オンプレミス環境の準備には２種類の実施方法がある:

- 準備済みの Virtual Box イメージのインポート
- 自分自身での環境構築

#### 自分自身での環境構築の方法

以下のソフトウェアをインストールする必要がある:


|ソフトウェア | インストール・パス | 備考|
|--- | --- | --- |
|JDK7 | /usr/java/jdk1.7.0_XX/ | VBox では JDK7u79 を使用したためパスは `/usr/java/jdk1.7.0_79/`|
|JDK8 | /usr/java/jdk1.8.0_XX/ | VBox では JDK8u60 を使用したためパスは `/usr/java/jdk1.8.0_60/`|
|Maven | /u01/wins/wls1221/oracle_common/modules/org.apache.maven_3.2.5 | その他の場所でもよい。環境変数 `PATH` に設定すること|
|リモートリポジトリのクローン場所| /u01/content/cloud-native-devops-workshop ||
|Weblogic Server 10.3.6 | /u01/wins/wls1036/ | VBox では、WLS 10.3.6.0.0 を使用|
|WebLogic Server 12.2.1 | /u01/wins/wls1221/ | VBoxでは、 WLS 12.2.1.0.0 を使用|
|Domain To Partition  Conversion Tool | /u01/dpct ||
|Oracle DataBase 12c | /u01/app/oracle/product/12.1.0/dbhome_1/ | PDB名: **PDBORCL**|

#### environment.properties の編集

Oracle Public Cloud 環境をスクリプトで操作するために、`<クローンしたGitリポジトリ>/cloud-utils` ディレクトリに含まれている `environment.properties` を構成する必要がある。

以下の2つのサンプル・ファイルを提供している:

- `environment.properties.us2`
- `environment.properties.emea2`

これをコピーし利用して、`environment.properties` を作成する。

このプロパティファイルの各変数について以下の説明する:

|プロパティ名|プロパティ値|備考|
| --- | --- | --- |
|**Account properties**|||
|opc.base.url|jcs.emea.oraclecloud.com|**EMEA** のデータセンター用の設定値。リージョンが **US** の場合は、`jaas.oraclecloud.com` を設定|
|opc.identity.domain|アイデンティティドメイン名|使用するアイデンティティドメイン名を設定|
|opc.username|ユーザ名|Oracle Cloud のユーザ名を設定|
|opc.password|パスワード|Oracle Cloud ユーザパスワードを設定|
|**Storage and container properties**|||
|opc.storage.generic.url|storage.oraclecloud.com|Storage CS のREST エンドポイント URL|
|opc.storage.container|Storage Contaiener 名|Cotainer 名のみを設定|
|**SSH key properties**|||
|ssh.user|opc|||
|ssh.public.key||Cloud Service のインスタンスの作成に使用する **公開鍵** の値を設定|
|ssh.passphrase||秘密鍵のパスフレーズを設定する。パパスフレーズを設定していない場合は、何も設定しない|
|ssh.privatekey||秘密鍵の名称を設定。`<クローンしたGitリポジトリ>/cloud-utils`に配置してある事に注意。 (VBox の場合: /u01/content/cloud-native-devops-workshop/cloud-utils)|
|**Java Cloud Service properties**|||
|jcs.base.url|jaas.oraclecloud.com||
|jcs.rest.url|/paas/service/jcs/api/v1.1/instances/||
|jcs.instance.1|JCS インスタンス名|Java Cloud Service インスタンス名|
|jcs.instance.admin.user.1|WebLogic 管理者名|JCS 定義中に設定する WebLogic 管理者ユーザ名|
|jcs.instance.admin.password.1|WebLogic 管理者パスワード|JCS 定義注に設定する WebLogic 管理者パスワード|
|**Database Cloud Service properties**|||
|dbcs.base.url|dbcs.emea.oraclecloud.com|**EMEA** のデータセンター用の設定値。**US** の場合は、`dbaas.oraclecloud.com` を設定|
|dbcs.rest.url|/paas/service/dbcs/api/v1.1/instances/||
|dbcs.instance.1|DBCS インスタンス名|Database Cloud Service インスタンス名|
|dbcs.instance.sid.1|ORCL|SID|
|dbcs.instance.pdb1.1|PDB1|Database Cloud Service インスタンスの作成時にデフォルト値として **PDB1** を使用|
|dbcs.dba.name|DBA 名|DBCS 定義中に設定する管理者名。多くの場合、**sys** を使用する|
|dbcs.dba.password|DBA パスワード|DBCS 定義中に設定する管理者パスワード|

#### Cloud Service インスタンスの作成

新規の Oracle Cloud アカウントを使用する場合、Storage Container、Database Cloud Service 及び Java Cloud Service インスタンスは作成されていない。そこで、`environment.properties` 内の以下3つの変数を編集する:
- **opc.identity.domain**
- **opc.username**
- **opc.password**

そして、次のスクリプトを実行する:

```
$ [oracle@localhost Desktop]$ cd /<クローンしたGitリポジトリ>/cloud.utils
$ [oracle@localhost cloud.utils]$ mvn install -Dgoal=generate-ssh-keypair
  (...)
$ [oracle@localhost cloud.utils]$ mvn install -Dgoal=jcs-create-auto
  (...)
```

これにより、Java Cloud Service 及び Database Cloud Service のインスタンスに必要な全ての要素が作成される。
`<クローンしたGitリポジトリ>/cloud-utils` に秘密鍵が生成され、`pk.openssh` と名前が付けられている事を確認する。この秘密鍵は、Cloud Service インスタンスの VM にアクセスする際に必要である。また、`environment.properties` ファイル内の `ssh.public.key` プロパティの値に公開鍵値が設定されている事も確認する。
