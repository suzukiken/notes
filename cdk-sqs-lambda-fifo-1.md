+++
title = "FIFO SQS + Lambdaの設定 実験1"
date = "2021-04-09"
tags = ["SQS", "Lambda"]
+++

[記事](/aws/cdksqs-lambda-fifo)の続き。実験1。

* バッチサイズ: 指定なし
* Lambda関数の同時実行数: 指定なし
* メッセージグループIDのバリエーション: 1

[CDKのコード](https://github.com/suzukiken/cdksqs-lambda-fifo)

## Lambda関数のタイムアウト

この実験ではバッチサイズの指定はしないので[AWSのドキュメント](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/with-sqs.html)によればLambdaのバッチサイズは最大で10となる。

実際AWSコンソールのSQSの一覧画面を何度もリロードしながら見ていると、inflightのメッセージの数は10をキープし続け、

![img](/img/2021/04/sqs-fifo-group-1-0.png)

可視メッセージは10ずつ減っていく様子が見れた。

![img](/img/2021/04/sqs-fifo-group-1.png)
![img](/img/2021/04/sqs-fifo-group-1-2.png)

これはCloudWatch Logsのログを見ても10ずつ消費されていることが確認できる。

![img](/img/2021/04/log-fifo-group-1.png)

だから、Lambda関数の処理にかかる時間は、1つのメッセージを処理するのにかかる時間の最低10倍必要になる。

そのためLambda関数の`timeout`は少し余裕を見て12秒を設定し、これで実際にテスト中にタイムアウトは起きなかった。

## Lambda関数の同時実行数

[AWSのブログ](https://aws.amazon.com/blogs/compute/new-for-aws-lambda-sqs-fifo-as-an-event-source/)を見るとこう書いてある。

> In SQS FIFO queues, using more than one MessageGroupId enables Lambda to scale up and process more items in the queue using a greater concurrency limit. Total concurrency is equal to or less than the number of unique MessageGroupIds in the SQS FIFO queue.

つまりメッセージグループIDが複数あるなら、Lambdaの同時実行数はメッセージに応じて、最大でメッセージグループの数まで増えてゆくことになるのだが、この実験ではメッセージグループIDが1つしか無いので、同時実行数の最大が1となる。

実際にAWSコンソールでLambdaのモニタリングを見ても確かに同時実行数は1のままだった。

![img](/img/2021/04/lambda-fifo-group-1.png)

## メッセージの不可視期間

これは、Lambda関数のタイムアウトより若干長めの15秒で問題なかった。

## 実験結果

300件のメッセージを投入する実験を3回試した。

3回とも全てのメッセージの処理が成功し、1件もDLQに移動されたメッセージはなかった。

処理にかかった時間は

* 302.074
* 301.785
* 301.591

900件のメッセージに対してLambda関数の呼び出しは93回だった。
        
ということは1回のLambda関数呼び出しにメッセージ処理以外に0.05秒程度の時間がかかっているが、
これはStandard SQSでの実験に比べると著しく短いことになる。（なんとなくデータが正しいか不安）