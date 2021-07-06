+++
title = "Amazon API Gateway でリダイレクト"
date = "2021-02-28"
tags = ["API Gateway"]
+++

![img](/img/2021/03/redirectapigw.png)

リダイレクトはS3やALBでもできるので、わざわざAPI Gatewayにリダイレクトを担当させるとしたら後ろにLambdaを置いて動的にリダイレクト先を変更したい場合だと思うのですが、今回は遊びなのでAPI Gatewayだけでリダイレクトするものを作ってみました。

その際にちょっと引っかかったところがあったので、それのメモを書いておきます。

[CDKのApi Gatewayに関するページ](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-apigateway.IntegrationResponse.html)の記事中の説明に

> You must enclose static values in single quotation marks

とあるのですが、これは例えば以下のように`https://www.google.com`をダブルクォートの中でさらにシングルクォートで囲む必要があるということ。

```TypeScript
responseParameters: {
    'method.response.header.Location': "'https://www.google.com'"
}
```

[Githubのリポジトリ](https://github.com/suzukiken/cdkapigwredirect)
