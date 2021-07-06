+++
title = "API Gatewayのログ"
date = "2021-02-28"
tags = ["API Gateway"]
+++

知らなかったけどApi Gatewayのログって出力できるんですね。

指定したロググループにはこのようなアクセスログが残る。

```
{
    "requestId": "d1924400-2fd9-4575-bdc0-3fbdfe3001f0",
    "ip": "13.114.207.117",
    "requestTime": "28/Feb/2021:00:34:18 +0000",
    "resourcePath": "/text",
    "status": "200"
}
```

`API-Gateway-Execution-Logs_<API ID>/<STAGE>`のロググループにはこうしたログが残る

```
API Key ***************apikey exceeded quota limit for API Stage xxxxx/prod: Key quota exhausted for Usage Plan ID xxxx. Limit: 200 Period: DAY
```

[Githubのリポジトリ](https://github.com/suzukiken/cdkapigwlogging)
