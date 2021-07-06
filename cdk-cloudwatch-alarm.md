+++
title = "CloudWatch AlarmでLambdaのスロットルを通知する"
date = "2021-03-31"
tags = ["CloudWatch Alarm", "Lambda"]
+++

例えばSQSでLambdaをトリガーしていて、Lambdaでスロットルエラーが起きている場合、CloudWatch LogsのLambda関数のログでそれがわからない。

そもそもそれを把握する必要があるか、というのはもちろん作るシステムにもよると思うけど、スロットルエラーが発生したらすぐに把握したいぞ、といった場合、CloudWatch Metricsのアラームで通知するという手がある、ということで作ってみた。

[Amazonのドキュメント](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)を読むと詳しく書かれているけど、設定のポイントとなるのは

* Period
* Evaluation Periods
* Datapoints

の3つのパラメータだろう。

今回作ったのはこの[リポジトリ](https://github.com/suzukiken/cdkcloudwatch-alarm)のものだけど、これは

* Period: 1（分ごとに）
* Evaluation Periods: （直近の）1（分間に）
* metric: スロットル（の発生が）
* Datapoints: 1（回以上なら）

メールでアラートを通知するという設定になっている。

あともう一つポイントとなるのが**データが無い部分をどう解釈するか**だ。

スロットルエラーの発生回数を把握する場合、逆にスロットルが発生していない期間はデータ自体が無い。

このデータが無い状態をどう解釈するか、の設定をデフォルトのままにしてしまうと、CloudWatchはデータが無いことをデータが得られないのだと解釈して「データ不足」というステータスになってしまう。

そうならないようにするには、データが無いことをCloudWatchが「スロットルが発生していないのだ」と解釈するように
`treatMissingData`のところを`NOT_BREACHING`にする必要がある。[参考](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)

実際に試してみると、一旦Alarm状態になった後、10分程度でOK状態に戻るという挙動になる。

[リポジトリ](https://github.com/suzukiken/cdkcloudwatch-alarm)のコードではこれを試してみるために、監視されるLambda関数の動作を

* [1秒停止](https://github.com/suzukiken/cdkcloudwatch-alarm/blob/master/lambda/monitored.py)
* [同時実行数1](https://github.com/suzukiken/cdkcloudwatch-alarm/blob/7e53238f62e0957823e7db62f9a929aa23fd779c/lib/cdkcloudwatch-alarm-stack.ts#L19-L28)

としておいて[10回非同期呼び出し](https://github.com/suzukiken/cdkcloudwatch-alarm/blob/master/test/invoke_lambda.py)をすることでスロットルを発生させるようにして試していた。

ところで、AWSのドキュメントを読んでいてわからなかったのは、evaluation rangeのことだった。

これがOK状態に戻る鍵になっていると思うのだがevaluation rangeの値が常に5（データポイント）なのか、それとも何かに応じて変化するものなのか、もし5に固定されているなら、なぜOKに戻るのに10分かかるのか、といった疑問が残った。

とはいえ10分だと困るわけでも無いので、まいっか。

ちなみに通常のAWSのサービスのデフォルト解像度（Resolution）は1分でだから、あまり迅速に問題の発生を把握できるわけではないと思う。これはカスタムでメトリクスを作れば1秒単位で設定することができるらしい。[参考](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html)


