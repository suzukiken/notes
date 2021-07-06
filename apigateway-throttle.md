+++
title = "Amazon API Gateway のApiKeyとスロットルとクォータ設定"
date = "2021-02-28"
tags = ["API Gateway"]
+++

![img](/img/2021/03/apigwthrottle.png)

[Githubのリポジトリ](https://github.com/suzukiken/cdkapigwplan)

試してみて気がついたこと。（自分の勘違いもあるかもしれない）

* [addUsagePlan](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-apigateway.RestApi.html#addwbrusagewbrplanid-props)はapiKey無しでデプロイできるが機能させるにはapiKeyの指定が必要。
* [addMethod](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-apigateway.Resource.html#addwbrmethodhttpmethod-integration-options)で`apiKeyRequired: true`しておきながら、ApiKeyをどのUsagePlanにも連携させなかった場合、そのAPIは呼び出しできない。

こんなふうに確認できる。
'''
curl -i -H "x-api-key: 12345678901234567890" <url>
'''