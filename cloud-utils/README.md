![](../common/images/customer.logo.png)
---
# ORACLE Cloud-Native DevOps workshop
-----
## Oracle Cloud 用ユーティリティ・ツール
ここで紹介するツールは、非公式のOracle Cloud 用のユーティリティ・ツールである。以下の Oracle Cloud Service に対して REST APIを使用してインスタンスの作成や削除を実施する Maven ベースのツールである。
この CLI ツールに関する情報は、以下のサイトで確認する事が出来る:

- [Oracle Java Cloud Service CLI](https://docs.oracle.com/cloud/latest/jcs_gs/jcs_cli.htm)

### 準備

#### 1. 無制限強度のJCS管轄ポリシーの準備

SSH 接続に **無制限強度のJCS管轄ポリシー** を必要とするため、以下のサイトからダウンロードする:

- [Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files 8 Download](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)

なお、JCE Unlimited Strength Jurisdiction Policy Files が JDK に含まれていないのは、特定の国々に対して高度な暗号化技術を輸出する事が米国輸出規制に制限されるためである。

ダウンロードし、展開すると JAR ファイルが2つ含まれているので、それらを `<JAVA_HOME>/jre/lib/security` にコピーする。

- local_policy.jar
- US_export_policy.jar

#### 2. 環境情報の準備

Oracle Cloud へアクセスするための情報を含むプロパティファイルを準備する。**environment.properties** を開く。デフォルトでは、Maven ビルドの中では、このファイル名 (environment.properties) で読み取るが、異なるファイル名の場合は、パラメータに次のようにして指定する事で実施可能である:

- ***-Dopc.properties=myproperties.props***

最低限以下の情報を設定する:

- **opc.identity.domain=<アイデンティティドメイン名>**
- **opc.username=<Oracle Cloud のアカウント名>**
- **opc.password=<Oracle Cloud のアカウントパスワード>**
- **ssh.passphrase=<秘密鍵のパスフレーズ>**

#### 3. SSH 鍵ペアの準備

公開鍵と秘密鍵のペアの生成を行う。

##### 3.1. 既に SSH 鍵ペア を持っている場合

既に SSH 鍵ペア (RSA, OpenSSH フォーマット) を持っている場合、次のプロパティ値に公開鍵の値をコピーする必要がある:

- **ssh.public.key=ssh-rsa AAAAB3NzaC1y........vaPU+vcppMHINDBAbPyT6qNIkl== rsa-key-20150227**

また、秘密鍵ファイルを `cloud-utils` ディレクトリにコピーし、**pk.openssh** に名前変更しておく。そして、秘密鍵のパスフレーズを次のプロパティに設定する:

- **ssh.passphrase=<秘密鍵のパスフレーズ>**

##### 3.2. ユーティリティ・ツールにより SSH 鍵ペアを生成する場合

ユーティリティ・ツールにより SSH 鍵ペアを生成する場合は、以下のプロパティのみ設定する:

- **ssh.passphrase=<秘密鍵のパスフレーズ>**

そして、次の Maven コマンドを実行する:

- **mvn install -Dgoal=generate-ssh-keypair**

これにより。秘密鍵ファイル (`pk.openssh`) が生成され、プロパティファイルの **ssh.public.key** 値が生成された情報により更新される。また、このビルドにより、既存の秘密鍵と公開鍵のバックアップが作成される:

- 秘密鍵: pk.openssh -> ***pk.openssh.現在の時間***
- 公開鍵: プロパティファイルから ***public.key.現在の時間***

なお、生成した秘密鍵を Putty で使用したい場合は、`puttygen` により Putty の PPK フォーマットに変換する必要がある。

#### 4.

DBA 及び WebLogic 管理者の認証情報がプロパティファイルには記載されているが、セキュリティのため変更する:

- **jcs.instance.admin.user.1=<WebLogic 管理者ユーザ名>**
- **jcs.instance.admin.password.1=<WebLogic 管理者パスワード>**
- **dbcs.dba.name=sys**
- **dbcs.dba.password=<DBAパスワード>**

プロパティファイルの準備が整ったら、Oracle Cloud の管理用の以下の Maven ゴールが使用できる:

- **generate-ssh-keypair**:
  - 現在の SSH 鍵ペア (秘密鍵と公開鍵) をバックアップし、新しいペアを生成する。パスフレーズは、プロパティファイルにに指定した情報を用いる。
- **jcs-create-auto**:
    - 全ての前提条件を含む　Java Cloud Service インスタンスをプロビジョニングする。まず、Storage Contaier、次に Database Cloud Service、最後に Java Cloud Service インスタンスをプロパティファイルの構成情報に基づいて作成する。既に同一コンポーネントが存在している場合は、ビルドに失敗する。このビルド作業には、1時間以上かかる場合があるので注意すること。
- **storage-list**:
  - 既存の Storage Container の一覧を表示する
- **storage-create**:
  - プロパティファイルに指定した名前で Storage Container を生成する
- **storage-get-details**:
  - Storage Container の詳細情報を表示する
- **storage-delete**:
  - プロパティファイルに指定した名前の Storage Container を削除する
- **dbcs-create**:
  - プロパティファイルに指定した構成で Database Cloude Service インスタンスを生成する
- **dbcs-get-instance-details**:
  - Database Cloud Service インスタンスに関する詳細情報 (パブリック IP アドレス、EM コンソール・アドレス、バージョン、ステータス、など) を表示する
- **dbcs-delete**:
  - Database Cloud Service インスタンスを終了する
- **jcs-create**:
  - プロパティファイルに指定指定した構成の Java Cloud Service を生成する
- **jcs-get-instance-details**:
  - Java Cloud Service インスタンスに関する詳細情報 (パブリック IP アドレス、WebLogic 管理コンソール・アドレス、バージョン、ステータス、など) を表示する
- **jcs-delete**:
  - Java Cloud Service インスタンスを終了する
- **jcs-get-job-details –Djob.id=X**:
  - Java Cloud Service インスタンスに関するジョブの詳細情報を表示する。例えば、インスタンスのプロビジョニングや終了のステータスなどを表示する。**X** は、**jcs-get-instance-details** の実行で得られるジョブ番号である。

Maven ビルドの実行フォーマットは、以下のようになる:

**mvn -Dgoal=GOAL** *-Dopc.properties=myproperties.props*

GOAL の箇所は、上記の一覧から選択する。*-Dopc.properties* は、プロパティファイルの指定である。デフォルトでは、***environment.properties*** が使用される。
