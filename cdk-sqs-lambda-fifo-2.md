+++
title = "FIFO SQS + Lambdaの設定　実験2"
date = "2021-04-09"
tags = ["SQS", "Lambda"]
+++

[記事](/aws/cdksqs-lambda-fifo)の続き。実験2。

* バッチサイズ: 1
* Lambda関数の同時実行数: 指定なし
* メッセージグループIDのバリエーション: 1

[CDKのコード](https://github.com/suzukiken/cdksqs-lambda-fifo)

## Lambda関数のタイムアウト

この実験ではバッチサイズを1に指定している。
AWSコンソールのSQSの一覧画面をリロードしながら見ていると、inflightのメッセージの数は1をキープし続ける。

![img](/img/2021/04/sqs-fifo-batch-group-1.png)

CloudWatch Logsのログを見ても1件ずつ消費されていることが確認できる。

![img](/img/2021/04/log-fifo-batch-group-1.png)

Lambda関数には1件のメッセージを処理するのに1秒かかるように書かれている。
そのため今回はLambda関数の`timeout`を1秒より多い2秒を設定し、これで実際にテスト中にタイムアウトは起きなかった。


## Lambda関数の同時実行数

[実験1](/aws/cdksqs-lambda-fifo-1)と同じように、この実験ではメッセージグループIDを全てのメッセージで同じ文字列にしているため、
同時実行数の最大は1となる。

実際にAWSコンソールでLambdaのモニタリングを見ても確かに同時実行数は1のままとなる。

![img](/img/2021/04/lambda-fifo-batch-group-1.png)

## メッセージの不可視期間

これは、Lambda関数のタイムアウトより1秒長い3秒にして問題なかった。

## 実験結果

300件のメッセージを投入する実験を3回試した。
3回とも全てのメッセージの処理が成功し、1件もDLQに移動されたメッセージはなかった。

処理にかかった時間は

* 317.186
* 315.031
* 315.721

900件のメッセージに対してLambda関数の呼び出しは900回だった。
[実験1](/aws/cdksqs-lambda-fifo-1)と同様、1回のLambda関数呼び出しの、
メッセージ処理以外にかかる時間が0.05秒程度となっているが、
これはStandard SQSでの実験に比べると著しく短い。

理由は知らない。