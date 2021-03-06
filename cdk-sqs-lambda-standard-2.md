+++
title = "Standard SQS + Lambdaの設定実験2"
date = "2021-04-08"
tags = ["SQS", "Lambda"]
+++

[記事](/aws/cdksqs-lambda-standard)の続き2。

* バッチサイズ: 1
* Lambda関数の同時実行数: 指定なし

[CDKのコード](https://github.com/suzukiken/cdksqs-lambda-standard)

### Lambda関数のタイムアウトについて

今回の実験ではLambda関数は最低1秒かかるようにしてある。その上でバッチサイズが1なので今回はタイムアウトを1より大きい2秒にしてタイムアウトは発生しなかった。

### Lambda関数の同時実行数について

実験では300のメッセージをSQSに一気（といっても1秒程度で）に投入して、Lambda関数のモニタリングで確認したが何度か試して同時実行数は最大で26になった。バッチサイズが小さいので[実験1](/aws/cdksqs-lambda-standard-1)に比べてこちらの方が多くの関数の実行環境が立ち上がるということなのだろう。

![img](/img/2021/04/lambda-standard-batch.png)

#### SQSのメッセージの不可視期間について

ここが不可解なのだが15秒に指定するのが良いと思われた。というのも（maxReceiveCountを1に指定している前提だが）この期間が短いとデッドレターが発生するためだ。

なぜ15秒必要なのか。AWSのドキュメントには根拠が見つからないので自分の推測なのだが、Lambda関数とSQSの間にいる何かがあって、これは例えばLambdaそのものかあるいはそれ以外か知らないが、それがバッチサイズの10倍程度のメッセージをSQSから取り込んでいるように見える。

これはAWSコンソールでSQSの「処理中のメッセージ」を何度も画面をリロードしながら見ていてそこが10程度になるのでそう考えた。

![img](/img/2021/04/sqs-standard-batch.png)

この取り込まれたメッセージがどうなるかは、これもAWSのドキュメントには根拠を見つけられないが、思うにLambdaが持つキューのような場所に取り込んで、それをLambda関数が1件ずつ処理しているのではないか。

キューといえばLambdaの非同期呼び出しの場合のことが思い浮かぶが、今回の処理の中ではスロットルは発生していないので、その扱い方はLambdaの非同期呼び出しの場合とは異なるということなのだろう。

ともかく不可視期間の設定としては、Lambda関数の1件の処理時間が1秒なので10のイベントを処理し終わるのが最速で10秒だからそれを少し増やした値を指定すれば良いのではないか。

[AWSのドキュメント](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/with-sqs.html)の中の説明で

> Lambda はバッチを最大 5 個まで読み込み、それらを関数に送信します。

> レコードの各バッチを処理する関数時間を許可するには、ソースキューの可視性タイムアウトを、関数で設定したタイムアウトの少なくとも 6 倍に設定します。関数が前のバッチの処理中に関数の実行がスロットリングされた場合、Lambdaの再試行のために追加時間が与えられます。

と書かれている内容が上に書いたことの説明になるのかもしれない。ただ5と書いているが自分のみた数字からは10でないと説明と一致しない。

5か10かはともかく、バッチサイズに応じてinflightなメッセージが増減するようではある。これは例えばバッチサイズを3とかに増やして実行してみると、inflightなメッセージの数が1の時より大きくなるということからそうなのだろうと考えた。

![img](/img/2021/04/sqs-standard-batch-5.png)

### 300件の処理にかかる時間

3回実行して300件全て処理が成功

処理にかかった時間

* 23.634秒
* 21.141秒
* 25.973秒

[実験1](/aws/cdksqs-lambda-standard-1)に比べてバッチサイズが1/10なのにこちらの方が処理が早い。同時実行数でこちらが優っていたためだろう。
