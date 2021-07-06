+++
title = "CloudFormationスタックの削除保護"
date = "2021-05-10"
tags = ["CloudFormation"]
+++

[CloudFormationスタックの削除保護](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/using-cfn-protect-stacks.html)を管理コンソールでアクティベートした場合にそのスタックはCDKで削除できるのか試してみた。

削除はできてしまう。何の問題もなく特にメッセージなどはなく削除できる。つまり削除保護はCDKには機能しない。