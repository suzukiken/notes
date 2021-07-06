+++
title = "Amazon API GatewayだけでIPアドレスを返す"
date = "2021-02-28"
tags = ["API Gateway"]
+++

![img](/img/2021/03/ipaddressapigw.png)

アクセス元のIPアドレスを返却するのを今までLambdaを使ってやっていたのですが、API Gatewayだけで作ることができるんですね。

[Githubのリポジトリ](https://github.com/suzukiken/cdkapigwipaddr)

Api Gatewayのmapping templateはVTLを使っていて、どんな変数が得られるのかはこちらのページを参考にしました。
* [API Gateway mapping template and access logging variable reference](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html)



