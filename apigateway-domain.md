+++
title = "Amazon API Gateway カスタムドメイン"
date = "2021-02-28"
tags = ["API Gateway"]
+++

![img](/img/2021/03/apigwdomain.png)

Amazon API Gatewayにカスタムドメインを設定するというだけ。

[Githubのリポジトリ](https://github.com/suzukiken/cdkapigwdomain)

デプロイすると https://greetings.figmentresearch.com でapiが提供される。

Stackを作るときにStackPropsにenvを与えて、リージョンを指定する必要がある。
[参考](https://docs.aws.amazon.com/cdk/latest/guide/environments.html)
