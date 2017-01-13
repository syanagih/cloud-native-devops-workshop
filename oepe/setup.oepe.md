![](../common/images/customer.logo.png)
---
# ORACLE Cloud-Native DevOps workshop
-----
## Developer Cloud Service を用いた Eclipse 統合開発環境 (Oracle Enterprise Pack for Eclipse) の利用

### 説明

Oracle Enterprise Pack for Eclipse (OEPE), Oracle JDeveloper, and NetBeans のような統合開発環境 (IDE) を使用し、Oracle Developer Cloud Service のプロジェクトにアクセスする事が可能である。Eclipse と OEPE は、Oracle Developer Cloud Service への統合機能を含んでおり、Developer Cloud Service で定義し公開する クラウドアプリケーション開発のタスクをIDEからアクセスする事が可能となる。

Developer Cloud Service と Eclipse no統合は以下の内容を含む:

- Developer Cloud Service のプロジェクトを表示する専用ビュー画面
- Mylyn と Developer Cloud Sercvicew の問題管理機能の統合
- Developer Cloud Sercvice の Git リポジトリとソース管理の統合

Eclipse は以下からダウンロードが可能である:

- [http://www.eclipse.org/](http://www.eclipse.org/)

また、 OEPE は以下からダウンロードが可能である:

- [http://www.oracle.com/technetwork/developer-tools/eclipse/downloads/index.html](http://www.oracle.com/technetwork/developer-tools/eclipse/downloads/index.html)

Eclipse を既に使用している場合は、Oracle Cloud Tools plugin を Eclipse Marketplace からダウンロードする事ができる。OEPE は、素手にプラグインがインストールされている。

### このチュートリアルについて

このチュートリアルは、以下を実施する:

- Eclipse (Oracle Enterprise Plugin for Eclipse) のセットアップ

### 前提

- チュートリアル:[Git リポジトリ用いた Developer Cloud Service プロジェクトの作成]((../springboot-sample/create.devcs.project.md) を実施し、Developer Cloud Service に SpringBoot アプリケーションのプロジェクトを作成しておく
- チュートリアル:[Developr Cloud Service を用いたApplication Container Cloud Service へのSpring Boot サンプル・アプリケーションのデプロイ](../springboot-sample/devcs.accs.ci.md) を実施し、Developer Cloud Service と Application Container Cloud Service を用いて継続的ビルドインテグレーションを作成しておく
- Eclipse IDE with Oracle Cloud Tools plugin をインストールした Eclipse 又は Oracle Enterprise Plugin for Eclipse を用意しておく

### 手順

#### Cloud Tools Plugin の構成

OEPE を開き、ワークスペースの場所を設定する (デフォルトのままでもよい) 。ウェルカムページを閉じる。上部メニューから **Window** -> **Show View** -> **Other** -> **Oracle Cloud** を選択する。

![](images/02.png)

**Oracle Cloud** を展開し、リストに含まれる **Oracle Cloud** を選択する。

![](images/03.png)

Developer Cloud Service に初回接続する際は、**Connect** リンクをクリックする。

![](images/04.png)

Oracle Cloud Service 接続ダイアログが表示されたら、以下の内容を入力する:

- **Identity Domain**: 使用する Developer Cloud Service のアイデンティティドメイン名
- **Username**: Oracle Cloud ユーザ名
- **Password**: 上記ユーザ名のパスワード
- **Connection Name**: Eclipse 上での Oracle Cloud への接続識別名。デフォルトでは、アイデンティティドメイン名が設定される。

![](images/05.png)

認証情報を安全に保管するためにマスターパスワードを設定する。設定すると、今後 OEPE を開く度に入力を求めれる事がなくなる。

![](images/06.png)

認証されたら、Developer Cloud Service にログインする。ログインすると、アサインされている Developer Cloud Service の全てのプロジェクトが表示される。以下の手順でプロジェクトツリーを開く:

- **設定した接続名** -> **Developer** -> **DevCSプロジェクト名** -> **Code**


開くと、Developer Cloud Service 上の Git リポジトリが確認できる。

![](images/07.png)

Git リポジトリをローカル PC にクローンするには、**ダブルクリック** 又は **右クリック** -> **Activate** を選択すると Git リポジトリのクローンが始まる。

![](images/08.png)

ローカルにクローンが作成されると、ワークスペースで使用できるようになる。

![](images/09.png)

Maven プロジェクトをインポートするには、左ペインの **Project Explorer** で右クリックし、ポップアップメニューから **Import projects** -> **Import...** を選択する。

![](images/10.png)

**Existing Maven Projects** を選択し、**Next** をクリックする。

![](images/11.png)

**Browse** でクローンした Git リポジトリに移動し、**springboot-sample** のにある **pom.xml** を選択する。

- ワークスペースディレクトリ配下にクローンしたディレクトリが存在

**Finish** をクリックする。

![](images/12.png)

OEPE は、プロジェクトのビルドを開始する。そして、Project Explorer にインポートされたプロジェクトが確認できる。

![](images/13.png)

#### 継続的インテグレーションを使用したコード変更とテスト実施

Developer Cloud Service にホストされているプロジェクトは、Git リポジトリに変更したコードをプッシュすると、アプリケーションのビルドと Application Container Cloud Service へのデプロイが実行されるジョブを作成した:

- チュートリアル:[Developr Cloud Service を用いたApplication Container Cloud Service へのSpring Boot サンプル・アプリケーションのデプロイ](../springboot-sample/devcs.accs.ci.md) にて実施

ブラウザを開き、Application Container Cloud Service にデプロイされいるサンプル・アプリケーションを確認する。

![](images/15.png)

OEPE へ戻り、Project Explorer 上で `springbootdemo` プロジェクトを開く:

- **Deployed Resources** -> **webapp** -> **WEB-INF** -> **views** -> **welcome.jsp**

![](images/16.png)

以下の部分を修正する:

```
<br>SpringBoot application demo. Current server time: <%= new java.util.Date() %> <br>
```

ページ上で変更が分かるように変更を加える。例:

```
<br>SpringBoot application demo. <font color="red">MODIFIED IN OEPE.</font> Current server time: <%= new java.util.Date() %> <br>  	
```

変更したら保存を行う。プロジェクトを右クリックし、**Team** -> **Commit...** を選択し、Gitに変更内容をコミットする。

![](images/17.png)

Git Staging view が表示される。まず、`welcome.jsp` を **Staged Changes** 領域に移動する。そしてコミットメッセージを入力する。オプションで、Git コミットの Author と Committer を入力する。

- Author: オリジナルのコードを書いた人
- Committer: コミットした人

最後に **Commit and Push...** をクリックする。

![](images/18.png)

プッシュ・ダイアログ上ではデフォルトブランチのまま、**OK** をクリックする。

![](images/19.png)

ブラウザに戻り、Developer Cloud Service で Build ページを開く。風呂度が実施されている事が確認できる。これは、Git リポジトリの変更をトリガーに起動されたものである。

![](images/20.png)

ビルドジョブが終了したら、デプロイタブに移動する。すると、新しくデプロイが開始されている事が確認できる。

![](images/21.png)

ブラウザから、Application Container Cloud Service にデプロイされたアプリケーションを確認する。変更内容が反映されている事が確認できる。

![](images/22.png)
