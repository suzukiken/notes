+++
title = "FIFO SQS + Lambdaの設定 まとめ"
date = "2021-04-09"
tags = ["SQS", "Lambda"]
+++

[記事](/aws/cdksqs-lambda-fifo)の続きで、最後のまとめ。

## 同時実行数1を指定することの問題

実験5と6のように同時実行数を1に指定したことが、悪い結果になったことは興味深い。
今回はFIFOで実験したが、先日スタンダードで実験した時も、同時実行数を1に指定した、
実験3と4はエラーが出るケースがあったことを考えると、SQSとの組み合わせでは同時実行数1の指定は避けるべきなのかもしれない。

同時実行数1を指定したいケースとして、例えば何らかの、呼び出し頻度に制限のあるAPIをLambdaから呼び出す場合が考えられるが、
そういった場合は、FIFOでメッセージグループIDのバリエーションを無くすという形で、目的を実現するべきなのかもしれない。

FIFOを使う目的として、順は不同でも良いので、1つ1つ処理していきたいということがあると思う。

これから開発するときにその辺りには注意をしたいと思う。

ただ、スタンダードとFIFOではSQSにメッセージを投げ込む際の
[スループットに大きな差](https://docs.aws.amazon.com/ja_jp/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-messages.html)
がある。秒3000件が可能な高スループットFIFOは2021-04の時点でまだプレビューの段階で、東京リージョンには導入されていない。

## 冪等性とかのことについて

そもそもバッチサイズが1より大である場合、Lambdaがいくつかのメッセージを処理した後に、
ある1つのメッセージの処理で失敗したら、そのバッチの中のメッセージは丸ごとキューに残って、
再度配信される。だからメッセージが何度も処理されることは十分にありうる。

なので、この先はバッチサイズを1にすることを前提に書くことにする。

Standard SQSはat-least-onceだが、
[AWSのドキュメント](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html#FIFO-queues-exactly-once-processing)に
にはFIFOはExactly-once processingと書かれている。
このExactly-onceは、SQSが同じメッセージを5分以内に受け取らない仕組みについて書かれているもので、
配信についてはどうなのかと思うのだが、

[AWSのブログ](https://aws.amazon.com/blogs/compute/new-for-aws-lambda-sqs-fifo-as-an-event-source/)には、

> However, it does not guarantee only once delivery when used as a Lambda trigger.

となっていて、これがどんなケースのことを言っているのかわからないが、
いずれにしても、実験6の設定で起きたような重複処理が起こるものだと考えれば、
AWSのブログにあるように、Dynamo DBのようなDBを使ってLambda側で冪等性を確保する必要があるのだろう。

## CloudWatch Logs Insightについて

今回の実験ではこんなクエリを使った

処理数（成功数、失敗数）の確認方法

```
filter @message like /action:[a-z]+:queue:[a-z0-9.]+:sendgroup:[-0-9T:]+ - /
| parse @message "action:*:queue:*:sendgroup:* - " as action, queue, sendgroup
| stats count(*) as cnt by queue, action, sendgroup
| display cnt, queue, action, sendgroup
| sort sendgroup desc
```

経過時間の確認方法

```
filter @message like /action:consumed:queue:[a-z0-9]+:sendgroup:20210404130632/
| (latest(@timestamp) - earliest(@timestamp)) / 1000 as duration
| display duration
```

Lambdaの関数呼び出し回数

```
filter @message like /START/
| stats count(*)
```
