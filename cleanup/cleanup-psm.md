![](../common/images/customer.logo.png)
---
# ORACLE Cloud-Native DevOps workshop
----
## PaaS Service Manager (PSM) コマンド・ライン・インターフェース を用いた Application Container Cloud Service インスタンスの削除

### チュートリアルについて

このチュートリアルでは、PaaS Service Manager (PSM) コマンド・ライン・インターフェース を用いて Application Container Cloud Service インスタンスを削除する。

### 前提

以下のチュートリアルを実施済み、または削除するインスタンスが存在する事:

- [Tomcat ベースのアプリケーションを Application Container Cloud Service へデプロイ](../accs-tomcat/README.md)

また、以下のチュートリアルで実施した PaaS Service Manager (PSM) コマンド・ライン・インターフェース のインストールがされている事
- [PSM コマンド・ライン・インターフェースを用いた Java Cloud Service のスケール・イン](../jcs-scale-psm/README.md)

### 手順

#### PSM CLI を用いた Application Container Cloud Service インスタンスの削除

ターミナルを開き、Application Containr Cloud Service にデプロイされているアプリケーション一覧を表示する。以下のコマンドを実行して、アプリケーション一覧を取得する:

- `psm accs apps`

```
$ psm accs apps
{
    "applications":[
        {
            "identityDomain":"shinyayjan",
            "appId":"2349df27-fec1-4b14-9009-b530416aa684",
            "name":"springboot",
            "status":"RUNNING",
            "createdBy":"shinyay",
            "creationTime":"2017-01-11T13:56:17.577+0000",
            "lastModifiedTime":"2017-01-11T15:15:03.727+0000",
            "subscriptionType":"HOURLY",
            "isClustered":false,
            "requiresAntiAffinity":false,
            "computeSite":"EM003_Z18",
            "instances":[
                {
                    "name":"web.1",
                    "status":"RUNNING",
                    "memory":"1G",
                    "instanceURL":"https://psm.europe.oraclecloud.com/paas/service/apaas/api/v1.1/apps/shinyayjan/springboot/instances/web.1"
                }
            ],
            "runningDeployment":{
                "deploymentId":"a88f9c87-4c21-4147-8103-280b69b26aac",
                "deploymentStatus":"READY",
                "deploymentURL":"https://psm.europe.oraclecloud.com/paas/service/apaas/api/v1.1/apps/shinyayjan/springboot/deployments/a88f9c87-4c21-4147-8103-280b69b26aac"
            },
            "lastestDeployment":{
                "deploymentId":"a88f9c87-4c21-4147-8103-280b69b26aac",
                "deploymentStatus":"READY",
                "deploymentURL":"https://psm.europe.oraclecloud.com/paas/service/apaas/api/v1.1/apps/shinyayjan/springboot/deployments/a88f9c87-4c21-4147-8103-280b69b26aac"
            },
            "appURL":"https://psm.europe.oraclecloud.com/paas/service/apaas/api/v1.1/apps/shinyayjan/springboot",
            "webURL":"https://springboot-shinyayjan.apaas.em3.oraclecloud.com"
        }
    ]
}
```

デプロイされているアプリケーション情報を取得すると、**name** プロパティにアプリケーション名が表示されている事が確認できる。このアプリケーションを PSM のコマンドを使用して削除を行う。まず、ヘルプでコマンドの情報を確認する:

- `psm accs help`

```
$ psm accs help

DESCRIPTION
  Oracle Application Container Cloud Service

SYNOPSIS
  psm accs <command> [parameters]

AVAILABLE COMMANDS
  o apps
       List all Oracle Application Container Cloud applications
  o app
       List an Oracle Application Container Cloud application
  o push
       Create or Update an Oracle Application Container Cloud application
  o scale
       Scale an Oracle Application Container Cloud Service instance for a...
  o delete
       Delete an Oracle Application Container Cloud application
  o stop
       Stop an Oracle Application Container Cloud application
  o start
       Start an Oracle Application Container Cloud application
  o restart
       Restart an Oracle Application Container Cloud application
  o logs
       List log details of all the instances of an Oracle Application Container...
  o log
       View log details of an instance of an Oracle Application Container Cloud...
  o get-logs
       Request for log details of an instance of an Oracle Application Container...
  o recordings
       View recording details of all the instances of an Oracle Application...
  o recording
       List recording details of an instance of an Oracle Application Container...
  o get-recordings
       Request for recording details of an instance of an Oracle Application...
  o operation-status
       View status of an Oracle Application Container Cloud application operation
  o help
       Show help
```

`delete`コマンドが一覧から確認できる。この `delete` コマンドのシンタックスをヘルプで確認する:

- `psm accs delete help`

```
$ psm accs delete help

DESCRIPTION
  Delete an Oracle Application Container Cloud application

SYNOPSIS
  psm accs delete [parameters]
       -n, --app-name <value>
       [-of, --output-format <value>]

AVAILABLE PARAMETERS
  -n, --app-name    (string)
       Name of the application

  -of, --output-format    (string)
       Desired output format. Valid values are [json, html]

EXAMPLES
  psm accs delete -n ExampleApp
```

ヘルプによるとアプリケーションの名前が必要な事が分かる。シンタックスとアプリケーション名が分かったので、アプリケーションを削除する:

- psm accs delete -n <アプリケーション名>

```
$ psm accs delete -n springboot
{
    "identityDomain":"shinyayjan",
    "appId":"2349df27-fec1-4b14-9009-b530416aa684",
    "name":"springboot",
    "status":"RUNNING",
    "createdBy":"shinyay",
    "creationTime":"2017-01-11T13:56:17.577+0000",
    "lastModifiedTime":"2017-01-11T15:15:03.727+0000",
    "subscriptionType":"HOURLY",
    "isClustered":false,
    "requiresAntiAffinity":false,
    "computeSite":"EM003_Z18",
    "instances":[
        {
            "name":"web.1",
            "status":"RUNNING",
            "memory":"1G",
            "instanceURL":"https://psm.europe.oraclecloud.com/paas/service/apaas/api/v1.1/apps/shinyayjan/springboot/instances/web.1"
        }
    ],
    "runningDeployment":{
        "deploymentId":"a88f9c87-4c21-4147-8103-280b69b26aac",
        "deploymentStatus":"READY",
        "deploymentURL":"https://psm.europe.oraclecloud.com/paas/service/apaas/api/v1.1/apps/shinyayjan/springboot/deployments/a88f9c87-4c21-4147-8103-280b69b26aac"
    },
    "lastestDeployment":{
        "deploymentId":"a88f9c87-4c21-4147-8103-280b69b26aac",
        "deploymentStatus":"READY",
        "deploymentURL":"https://psm.europe.oraclecloud.com/paas/service/apaas/api/v1.1/apps/shinyayjan/springboot/deployments/a88f9c87-4c21-4147-8103-280b69b26aac"
    },
    "currentOngoingActivity":"Deleting Application",
    "appURL":"https://psm.europe.oraclecloud.com/paas/service/apaas/api/v1.1/apps/shinyayjan/springboot",
    "webURL":"https://springboot-shinyayjan.apaas.em3.oraclecloud.com",
    "message":[]
}
Job ID : 3619562
```

削除ジョブが正常にサブミットされた。Application Container Cloud Service インスタンスの削除には数分かかる。削除の結果は、Application Container Cloud Service の コンソール画面、又は PSM コマンド `psm accs apps` を使用して確認を行う事ができる。
