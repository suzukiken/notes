+++
title = "S3に置いた静的サイトをCloudFrontで配信する"
date = "2021-03-14"
tags = ["CloudFront", "S3"]
+++

指定したドメイン名でCloudFrontを使ってS3のコンテンツを配信するというだけ。

[Githubのリポジトリ](https://github.com/suzukiken/cdkcloudfronts3)

### CDKがデプロイするもの

* S3バケット
* それを配信するCloudFrontの設定
* カスタムなドメイン名でそれを閲覧できるようにするCloudFrontの設定とRoute53のレコード

デプロイするとhttps/httpどちらでもbonsoir.figmentresearch.comでコンテンツが提供される。

### メモ

Stackを作るときにStackPropsにenvでリージョンなどを与える必要がある。
[参考](https://docs.aws.amazon.com/cdk/latest/guide/environments.html)

Cache Policyを指定していないので、デフォルトの86400秒（1日）がキャッシュ保持期間になる。

動作の確認にはそれだと不便だったりするので、確認用のオブジェクトのヘッダには`Cache-Control: max-age=1`をつけてそのオブジェクトだけキャッシュを1秒にするといったことができる。
[参考](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Expiration.html#expiration-individual-objects)

S3のCORSの設定は必須ではない。
