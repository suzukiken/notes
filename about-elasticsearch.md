+++
title = "AWS Elasticsearch Service バージョン2.3について"
date = "2021-05-27"
tags = ["Elasticsearch"]
+++

Elasticsearchについてはいろいろ以前考えて結局使うならAWSのElasticsearch Serviceの古いバージョンを使うことにした、というのは前の記事に書いた通りで、一応理由もそこに書いた。

簡単に言えば基本的には勉強用だし実用的な意味でも自分が使う極小のデータなら用が足りそうだからだ。

で、そのバージョンが2.3なので今時のElasticsearchとはいろんな意味で違いがあって、ウェブのドキュメントを見てもそのままでは使えないことがある。

またAWS Elasticsearch Serviceとは別にElastic.coのElasticsearchもあり、Elastic.coの情報の方が検索にはよく登場するのでそちらのドキュメントを見ることも多い。もちろん自分でElasticsearchをインストールして動かすこともできるので、それをしている人の記事も検索に出てくる。

いろんな情報の中で、自分の使っているElasticsearchはどう違うのか、といったことをメモ的に書いておかないと忘れてしまうので、この記事はそういう目的で書くことにした。

### stringからtextとkeywordに

stringというタイプはElasticsearchのバージョン5あたりで無くなり、代わりにtextとkeywordが登場した。

https://www.elastic.co/blog/strings-are-dead-long-live-strings

textはAnalyzeされる。Inverse Indexが作られる。
keywordはAnalyzeされない。

自分が使うのは2.3なのでウェブなどを参考にする際はstringに置き換えて考える必要がある。

### Elasticsearchのバージョンは2.4の次が5.0

今のElasticsearchの最新バージョンが7.13で、自分が使っている2.3はとても古い感じがするし実際古いのだと思うけど3.xや4.xがあるわけではないから、メジャーバージョンは2,5,6,7と進んだわけで乖離は3つということ人ある。

### マッピングタイプ

6以降では1つのインデックスの中に1つのドキュメントタイプしか作ることができなくなったが、以前は作ることができた。
だから2.3では指定が可能だけど仕事などで新しいバージョンのElasticsearchを利用していることもあって、2.3でもタイプは1つしか作らないことにする。

https://www.elastic.co/guide/en/elasticsearch/reference/6.8/removal-of-types.html
にはマッピングタイプを辞めた理由が書かれている。

この記事に書かれているdocument typeというのは何かというと、これはよくわからない。

自分の解釈なので間違っていたら申し訳ないがどうやらこういうことらしいと解釈している。

document typeはマッピングタイプみたいなもの。もちろんマッピングタイプは廃止になったので、今時のElasticsearchのインデックス内には1つしかdocument typeを持つことができない。
[メタフィールド](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-fields.html)の`_type`はそのdocument typeの名前が設定されている。
今日のElasticsearchのバージョンでは`_type`は常に`_doc`である。

Elasticsearch 2.3では`doc_type`を`_doc`とするとエラーとなる。

### analysis-kuromojiプラグインは入っている

AWS Elasticsearch Service 2.3は日本語の全文検索でよく利用されるKuromojiを形態素解析に利用できる。

例：
https://github.com/suzukiken/elasticsearch-example/blob/master/python/es_create_index_and_mapping.py

### テーブルとか行とか列の呼び方

Mysql | Elasticsearch
--- | ---
テーブル | インデックス
行 | ドキュメント
列 | フィールド

### 検索の重みづけ

重みづけの方法としてはboostを利用することができる。

https://github.com/suzukiken/elasticsearch-example/blob/4ca4fbf3d721580609db4b06d5879cb09f3ccaf0/python/es_search_document.py#L148

### 2.3のドキュメント

https://www.elastic.co/guide/en/cloud-enterprise/2.3/index.html