+++
title = "Standard SQS + Lambdaの設定 メモ"
date = "2021-04-08"
tags = ["SQS", "Lambda"]
+++

[記事](/aws/cdksqs-lambda-standard)の続き5

4種類の実験をして分かったことやその実験に使ったCloudWatch Logs Insightについてまとめる。

## SQSとLambdaの関係は謎なところがある

実験3の不可視メッセージ数と不可視タイムアウトとリトライ設定については、なぜそうなるのか疑問のままなので、何かの方法で確認したいと思う。わかったらまた記事を更新したいと思う。

## 冪等性とかのことについて

そもそも今回の実験で使ったFIFOでないスタンダードのSQSは、
[メッセージの複数のコピーが順不同で配信される](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues.html)
ことがある。（at least once）

だから冪等性はLambda側で確保する必要がある。

実際に今回の実験では合計900 x 4 = 3600件を処理した範囲では、投入した件数以上のログは見つからないので、簡単に発生するものではないと思うが、もし1回だけ（exactly once）にしたい場合はSQSはFIFOを使う必要がある。

またLambdaが処理する際の動作としては、一つのバッチの中に複数のメッセージがある場合、バッチの中のメッセージの一つの処理でLambda関数がエラーとなった場合、そのバッチのメッセージは全てSQSに残り再度配信される。

だからバッチサイズを1より大きくした場合には、同じメッセージが何度も処理されることが比較的簡単に起こる。

これはランダムにエラーを起こすLambda関数を作ってバッチサイズ10などにして実験すれば、すぐに同じメッセージを何度も処理しているログが見つけられる。

こうしたこうしたことを考えて設計しておかないとならない。

## CloudWatch Logs Insightについて

今回はほぼ初めてまともにCloudWatch Logs Insightを使ってとても便利だった。

今回の実験で使った[コード](https://github.com/suzukiken/cdksqs-lambda-standard)で試す場合は、こういったクエリで件数や処理にかかった時間を確認した。

処理数（成功数、失敗数）の確認方法

```
filter @message like /action:[a-z]+:queue:[a-z0-9]+:sendgroup:[-0-9T:]+ - /
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

なおCloudWatch Logs Insightで集計できるが、最初の1回目は正しく集計されないという経験をしたので、その点は注意が必要。どういう理由なのかは知らない。
