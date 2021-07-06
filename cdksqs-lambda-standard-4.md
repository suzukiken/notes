+++
title = "Standard SQS + Lambdaの設定実験4"
date = "2021-04-08"
tags = ["SQS", "Lambda"]
+++

[記事](/aws/cdksqs-lambda-standard)の続き4。

| パラメーター | 設定 |
|----------|------|
| バッチサイズ | 1    |
| Lambda関数の同時実行数 | 1    |

の設定で実験した。

[CDKのコード](https://github.com/suzukiken/cdksqs-lambda-standard)

Lambda関数のタイムアウトはバッチサイズが1なので、1件のメッセージの処理にかかる時間（実験では1秒）に足りれば良いので、2秒に設定して実際にタイムアウトは発生していない。

Lambda関数の同時実行数は1に指定していて、実際にモニタリングで見ても1を維持していた。

![img](/img/2021/04/lambda-standard-mixed.png)

#### SQSのメッセージの不可視期間について

バッチサイズの10倍程度の不可視メッセージが常に存在していた。

![img](/img/2021/04/sqs-standard-mixed.png)

実験3と同様ここもよくわからないところなのだが、不可視タイムアウトは100程度にしないとエラーが発生する。
10件のメッセージの処理にかかる時間は、15秒程度で良いのではないかと思うのだが、それより数倍は多い数字を不可視タイムアウトに設定することでかなりエラーを減らすことができた。

実際には100秒にしていても相変わらずエラーは時々出る。

#### 300件の処理 x 3回の実験結果

|          | 回数 | 内訳              |
|----------|------|-------------------|
| 処理成功 | 2    | 300件全て         |
| 処理失敗 | 1    | 298件成功、2件DLQ |

処理にかかった時間

* 430.757秒
* 434.807秒
* 449.822秒

[実験3](/aws/cdksqs-lambda-standard)に比べると100秒近く余計に時間を食うことがわかった。

300件 x 3回 = 合計900件のメッセージの処理をしたとして計算すると、平均的に1回のLambda関数実行のために約0.5秒程度の余計な時間がかかっているということになる。これは実験3に比べると半分の時間だが、関数の呼び出し回数がこちらは800回程度多いので結果的には大きな差が生まれた。