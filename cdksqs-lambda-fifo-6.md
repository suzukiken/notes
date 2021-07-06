+++
title = "FIFO SQS + Lambdaの設定 実験6"
date = "2021-04-09"
tags = ["SQS", "Lambda"]
+++

[記事](/aws/cdksqs-lambda-fifo)の続き、実験6。

* バッチサイズ: 1
* Lambda関数の同時実行数: 1
* メッセージグループIDのバリエーション: 5

[CDKのコード](https://github.com/suzukiken/cdksqs-lambda-fifo)

## Lambda関数のタイムアウト

この実験ではバッチサイズの指定を1にしている。
またLambda関数は1つのメッセージを処理するのに1秒かかるように作ってあるので、
タイムアウトは1秒以上あれば良いと考えて余裕を見て2秒にしておいた。
これで実験中にタイムアウトは発生していない。

## Lambda関数の同時実行数

メッセージグループが5つあるが、Lambda関数の同時実行数は指定した通り1となる。

![img](/img/2021/04/lambda-fifo-mixed-batch-5.png)

## 不可視メッセージ数

バッチサイズが1でメッセージグループが5つあるので、inflightのメッセージの数は5となるということだろう。

バッチサイズ:1 x メッセージグループ数:5 = 不可視メッセージ数:5

![img](/img/2021/04/sqs-fifo-mixed-group-5.png)

ただ、同時実行数が1に固定されているため処理待ちが発生するのだろう、スロットルが発生していることが、
Lambda関数のモニタリング結果から見れる。

## メッセージの不可視期間

この実験ではmaxReceiveCountを1にしているので、[実験5](/aws/cdksqs-lambda-fifo-5)と同様の理由で、
タイムアウト時間を決めるために何度か設定を変更して、デッドレターの件数を確認した。

| 不可視タイムアウト(秒)  | デッドレター件数 |
|-------------------------|------------------|
| 3                       | 230              |
| 30                      | 20               |
| 60                      | 13               |
| 100                     | 5                |
| 120                     | 0                |
| 120                     | 3                |
| 110                     | 2                |
| 120                     | 2                |
| 120                     | 1                |

この結果から、以降の実験では120秒を採用した。

## 実験結果

300件のメッセージを投入する実験を3回試した。
メッセージグループは5つで、それぞれ均等に60件ずつのメッセージが同じ1つのメッセージグループとなっている。

3回全て、最低1件のメッセージがDLQに移動された。

| 成功 | デッドレター |
|------|--------------|
| 299  | 1            |
| 299  | 1            |
| 298  | 2            |


成功したメッセージの処理にかかった時間は

* 359.256秒
* 357.123秒
* 367.446秒

[実験5](/aws/cdksqs-lambda-fifo-5)よりもさらに遅い。

実験5と同様に[実験1](/aws/cdksqs-lambda-fifo-1)と比較するとこうなる。

|               | Lambda関数の同時実行数の指定 | メッセージグループIDのバリエーション | バッチサイズ | 処理にかかった平均時間（秒） |
|---------------|------------------------------|--------------------------------------|--------------|------------------------------|
| 実験1         | 指定しない                   | 1                                    | 指定しない   | 301.82                       |
| 実験6（今回） | 1                            | 5                                    | 1            | 361.275                      |

実験1では同時実行数をコードでは指定していないが、実際にはメッセージグループIDが1種類しかないので、同時実行数は1となる。
今回の実験6は、メッセージグループIDが5種類あるので、本来ならLambda関数の同時実行数は5までスケール可能だが、そこを1に制限している。
その結果として、今回の実験は処理に時間がかかっただけでなく、エラーまで出たので、良いことがない。

結局、Lambda関数の同時実行数を制限したいなら、メッセージグループIDを常に同じにして、
同時実行数の指定の方はしないほうがいいということなのではないか。

## ある現象

ついでに書いておくと、この実験6の設定で、何度か実験をしているときに、メッセージの重複処理のようなものが見られた。
重複処理といっても、あくまでCloudWatch Logsのログ上から得られる情報で見ると、それが見て取れるという話で、
そのログの出どころはLambda関数のコードの最後の行に書かれたprint文だ。

|                              | タイムスタンプ                | メッセージ                                                                              | バッチサイズ | 処理にかかった平均時間（秒） |
|------------------------------|-------------------------------|-----------------------------------------------------------------------------------------|--------------|------------------------------|
| ソースキューの処理ログ       | 2021-04-09T10:52:27.328+09:00 | action:consumed:queue:cdksqslambdafifomixed.fifo:sendgroup:20210409014836:group:e - 105 | 指定しない   | 301.82                       |
| デッドレターキューの処理ログ | 2021-04-09T10:52:26.605+09:00 | action:rejected:queue:cdksqslambdafifomixed.fifo:sendgroup:20210409014836:group:e - 105 | 1            | 361.275                      |

タイムスタンプから見ると、デッドレター処理の方が先に発生している。
ということは、元のメッセージはLambda関数が処理したのだが、その次のアクションとしてSQSのメッセージの削除がなされる前に、
不可視タイムアウトが発生して、メッセージはデッドレターに回された、ということなのかと想像している。