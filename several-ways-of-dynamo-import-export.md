+++
title = "Dynamo DBのインポートエクスポートいろんな方法の違い"
date = "2021-03-06"
tags = ["Dynamo DB", "DataPipline", "S3", "EMR"]
+++

この記事は2021年3月頭の時点の情報ですし、ここに書いた以外の方法もあるかもしれません。という前置きを書いて本題に。

MySQLだと`mysql dump`でエクスポートしたsqlファイルをインポートするといったことができる。

Dynamoでもそれと同じようなことができるのだろうか。
キャパシティーの設定もあるので、一気に大量のデータの出力、入力ができるのか。
できるとしてもどんな費用で何時間かかるのか。

その辺りのことについて勉強したことをまとめておく。

## いくつかの方法

少なくともこうした方法がある。

1. Dynamo DBのエクスポート機能を使って出力し、そのデータを加工してAPIを使ってインポートする。
2. DataPiplineでS3へデータをエクスポート、インポートする。
3. EMRを使う（AWSのドキュメントにあったので載せてるけどそもそもEMRって何？）
4. Dynamo DBのPoint-in-time recovery (PITR)機能で作ったバックアップを、新しいテーブルとして復元する。
5. Dynamo DBの削除時に作ったバックアップを復元する。

それぞれについてわかっている範囲で説明を残しておく。

### Dynamo DBのエクスポート機能

Dynamo DBのエクスポート機能は2020年11月頃のブログで紹介されてもので、比較的新しい機能なのだと思う。

AWSコンソールのボタンを押すだけなのでとても手軽に使える。
エクスポート先はS3の指定した場所。
APIで実行することもできる。

例えば75,000件、45MBのテーブルのエクスポートをしてみたところ2-3分で完了した。
これは後述するDataPiplineを使うのに比べればはるかに早い。

アウトプットは指定したディレクトリにmanifestファイルなどとともに吐き出される。

![img](/img/2021/03/dynamo-export.png)

データそのものは`.json.gz`の拡張子が付いたファイルで、gunzipするとDynamo DBの1アイテムが、
1行のjsonとなったものになる。（つまりファイル全体としてはjsonの配列になっているわけではない）

```json
{"Item":{"Id":{"S":"order-507-0343105-877700"},"address":{"M":{"city":{"S":"墨田区"}}}}
{"Item":{"Id":{"S":"order-245-0343070-862268"},"address":{"M":{"city":{"S":"横浜市緑区"}}}}
{"Item":{"Id":{"S":"order-102-0343063-517275"},"address":{"M":{"city":{"S":"白井市"}}}}
```

エクスポートしたデータはPutItemやBatchWriteItemなどのAPIでインポートできる。
ただし、エクスポートしたデータをそれらのAPIで処理できる状態に加工する必要がある。

[AWSのドキュメント](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DataExport.html)には、
データの用途として、Athena、Glue、Lake Formationが挙げられている。
つまりDynamo DBにインポートすることを前提とした機能ではないということだと思う。

[こちらの記事](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBPipeline.html)には、
DataPiplineのインポートには使えないと注意書きが書いてある。
実際、DataPiplineでエクスポートしたものとはフォーマットが異なる。

というわけであえてインポートをするなら、例えば
[AWSのドキュメント](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SampleData.LoadData.html)
に出てくるサンプルデータのような形に加工して、BatchWriteItemでデータを投入するのが、近道なのかなと思う。

つまりDynamo DBのエクスポート機能はとても便利だが、インポートの方法は特に用意されていないということだ。

### DataPipline

Dynamo DBのエクスポート、インポートの方法として、DataPiplineを紹介した記事が検索では良く出てくるという印象はあるのだが、
以下に書く通りAWSはそれを推奨してはいないし、確かに自分もできれば他の機能で済ませたいという印象を持っている。

この方法は[上のURL](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBPipeline.html)には、

> We highly recommend that you use DynamoDB's native backup and restore feature instead of using AWS Data Pipeline.

ということで、つまりおすすめされていない。
後述するDynamo DBのバックアップ・リストア機能を使う方がおすすめだそうだ。

とはいえ、DataPiplineの利点もある。

それはエクスポートしたデータがファイルとして扱えるということだ。
そのファイルを別のAWSアカウントのDynamo DBに持っていってインポートするといったことができる。

ところでDataPiplineはAWSコンソールのUIがわかりにくいという印象があるが、
Dynamo DBとS3のデータやり取りについては、予め用意されているフォームにテーブル名などを設定するだけなので、
とても手軽にできる。

S3へのアウトプットは指定したディレクトリにmanifestファイルなどとともに吐き出される。

![img](/img/2021/03/dynamo-pipline-export.png)

データの構造は前述のエクスポートの機能とは異なるが、やはり1アイテム毎に1行のJsonになっている。
```
{"Id":{"s":"order-507-0342852-435405"},"address":{"m":{"city":{"s":"横浜市緑区"}}}}
{"Id":{"s":"order-245-0343097-999727"},"address":{"m":{"city":{"s":"横浜市保土ケ谷区"}}}}
```

ちなみにDataPiplineのUIがわかりにくいのは、とても汎用的なサービスだからなのだろう。
[GetPipelineDefinition](https://docs.aws.amazon.com/datapipeline/latest/APIReference/API_GetPipelineDefinition.html)
で見るとS3にデータをエクスポートするパイプラインの中身はかなり複雑な設定であることがわかる。

この方法はDynamo DBのキャパシティーを消費する。
エクスポート、インポートのパイプラインを作る時に、
キャパシティーをどれだけ消費して良いか（ReadThroughputRatio、WriteThroughputRatio）を指定するようになっている。

実験として、75,000件、45MB、読込・書込キャパシティーそれぞれ1のテーブルで、
ReadThroughputRatio・WriteThroughputRatioをそれぞれ1として試すとこうなる。

* エクスポート：2時間
* インポート：21時間

Dynamo DBのキャパシティーは指定した通り目一杯消費している。

![img](/img/2021/03/dynamo-consumed-capaticy-read.png)

つまりRDSに比べると気が遠くなるような時間がかかってしまう。

多分使い方としては、こうした処理をする最中だけ100倍ぐらいにプロビジョンキャパシティーを拡大しておくべきなのかもしれない。
Read/Writeのキャパシティーを100にしてみたところ

* エクスポート：9分（見方がよくわからないが、実際にエクスポートが実行されている時間は多分3分）
* インポート：20分（実際にエクスポートが実行されている時間は多分14分。）

100を目一杯消費している。

![img](/img/2021/03/dynamo-consumed-capaticy-write.png)

この方法を使う場合に注意なのは、キャパシティーを減少させる回数は1日4回までとなっていること。

[Service, Account, and Table Quotas in Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Limits.html)

> A decrease is allowed up to four times, anytime per day.

なおオンデマンドキャパシティーだった場合にはどうなるのか、そのうち試したら追記する。

### EMR

別のAWSアカウントにデータを持っていく方法として[AWSのページ](https://aws.amazon.com/jp/premiumsupport/knowledge-center/dynamodb-cross-account-migration/)には、
DataPiplineとEMRが挙げられている。DataPiplineよりEMRの方が自由度が高いとのことだ。

そもそも上に書いたDataPiplineの処理は内部的にはEMRを使っているようなので、直接EMRを使う方が色々できますよ、ということなのだろう。

[ドキュメント](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/EMRforDynamoDB.html)を見すると確かにデータの加工ができるのかなという印象を持った。
つまりスキーマの変更をしたいといった場合に使ったりするのかもしれない。

EMRを使う方法は試していないので、なんとも言えないが、とにかくそういう手段があるということだ。

### Dynamo DBのバックアップ、リストア機能

先に結論をいうと、基本的にこれで済むことはこれで済ませるべきだ。

AWSコンソールを使うならボタンをポチるだけで、バックアップは一瞬でできる。

例えば75,000件、45MBのテーブルのバックアップ・リストアをしてみたところ、
バックアップには1秒かからず、リストアは1分程度だった。

費用はストレージに対してかかる費用と、復元時はデータサイズに対する費用がかかるが、
**キャパシティーは消費されない**。
（[参考](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/BackupRestore.html)）

バックアップされたデータは元のテーブルを削除しても残っているので、
削除したのと同じテーブル名で復元することもできる。
もちろん異なる名前のテーブルに復元することもできる。

ただし、どこかにファイルが残ってそれを取り出せるわけではない。
それから、既存のテーブルに対してバックアップ内容をインポートすることはできず、
常に新しいテーブルを作る形で復元することになる。

日時を指定したバックアップを利用するにはそのテーブルのPITR機能をオンにしておく必要がある。
テーブルを削除する際にバックアップを残すだけならその必要はない。

ところでCDKでTableを作るときの設定で`cdk destroy`時に、
バックアップを取ってから削除されるような指定できたらいいなあと思うけど、
removalPolicyで`SNAPSHOT`をこんなふうに指定してみると
```
const table = new dynamodb.Table(this, "table", {
  ...
  removalPolicy: cdk.RemovalPolicy.SNAPSHOT,
  ...
})
```
利用できない旨のメッセージが出る。
```
Template error: resource type AWS::DynamoDB::Table does not support deletion policy Snapshot
```
CDKでこうしたデータ保存のリソースをどう扱うべきかは悩みどころで、自分の中でこれ、という方法は見つけられていない。

## まとめ

1. Dynamo DBのエクスポート機能：便利だがインポートには向かない
2. DataPipline：他の環境に移行する時などに使う
3. EMRを使う：データの加工をしたい場合に使う
4. Dynamo DBのPITR機能：最高に便利だがダンプファイルは得られない
5. Dynamo DBの削除時のバックアップ：最高に便利だがダンプファイルは得られない