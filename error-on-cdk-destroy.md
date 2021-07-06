+++
title = "cdk destroy時のエラーについて"
date = "2021-05-09"
tags = ["CDK"]
+++

複数のスタックで同じリソースを利用しているとして、そのリソースを作ったスタックを`cdk destroy`するとエラーが出て削除できないことがある。

例えばこれはSSL証明書をACMで作ったスタックを`cdk destroy`しようとして、その証明書をどこかで使っている場合に表示されるエラーメッセージ。

> Received response status [FAILED] from custom resource. Message returned: Response from Certificate did not contain an empty InUseBy list after 10 attempts.


こんな風に利用中のSSL証明書は削除できないのに対して、例えばAppSyncのデータソースとして使っているDynamoDBのテーブルを作ったスタックはAppSyncを作ったスタックを残したまま削除できてしまう。

その結果どうなるかというとAppSyncのAPIを呼び出したときにエラーが出る。

SSL証明書のように`cdk destroy`ができないものとAPIで利用しているテーブルのように削除できるものの違いはどこから来るのか、今後ちょくちょく調べてゆきたい（けどやらないかもしれない）。