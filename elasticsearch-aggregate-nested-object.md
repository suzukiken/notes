+++
title = "Elasticsearchでネストを展開して集計する"
date = "2021-03-20"
tags = ["Elasticsearch"]
+++

実はElasticsearchはもう使わないことにしたのだけど、以前こんなことができるのかなと試したことをまだ記憶があるうちにメモっておくのがこの記事の目的だ。

何を試したかというとElasticsearchで配列を展開して条件に合う行だけ集計するというのをやってみた。

![img](/img/2021/03/elasticsearch-aggregate-nested-object.png)

Elasticsearchには配列を持つJsonを突っ込んで、その配列の中の特定の条件に一致する行についてだけ集計するということをしてみている。

[コード](https://github.com/suzukiken/cdkelasticsearch/tree/master/test)

* Elasticsearchのマッピング（テーブル定義みたいなもの）を指定
* インデックス（テーブル）の作成
* ドキュメント（レコード）の投入
* クエリ
* 削除

というのをそれぞれpythonのファイルに分けて書いた。

ちなみに認証はrequests_aws4authを使っている。[参考](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-request-signing.html)