+++
title = "AWSコンソールでCloudFormationテンプレートを編集する"
date = "2021-05-09"
tags = ["CloudFormation"]
+++

Systems Managerを介してスタック間で値を共有している場合、パラメータを[動的な参照](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/dynamic-references.html)しているスタックがあってもパラメータは削除できるし、そのパラメータを作ったスタックも削除できる。

ただ削除されたパラメータを[動的な参照](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/dynamic-references.html)しているスタックを`cdk deploy`するとこうしたエラーが出てデプロイができなくなる。

> BStack failed: Error [ValidationError]: Parameters: [ssm:/stack/a/output] cannot be found.

これは同じ名前のSystems Managerのパラメータを新しく作れば解決できるし、もしSystems Managerのパラメタのバージョンを指定してあってもそのバージョンまでSystems Managerのパラメータを更新して要を参照できるようにしてやればまたデプロイできるようになる。

ま、それをやればいいのですがそれはそれで手間だったりする。

そこで違うアプローチとして取りうる方法はAWSコンソールでCloudFormationテンプレートを編集する方法で、もちろんAWSコンソールではなくCLIなどでもできると思うけど自分はやったことがないので知らない。

ともかく`cdk deploy`ができなくてもAWSコンソールでCloudFormationテンプレートを編集してそれをデプロイすることはできる。

例えばあるDynamo DBのテーブル名を[動的な参照](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/dynamic-references.html)で得ていたとする。

CloudFormationテンプレートはこんな風になっていると思う。

バージョンの指定をしていない場合
> "TableName": "{{resolve:ssm:/xx/xx/xx/tablename}}"

バージョン2の指定をしている場合
> "TableName": "{{resolve:ssm:/xx/xx/xx/tablename:2}}"

この`{{resolve:ssm:/xx}}`のところをテーブル名に変更する。

> "TableName": "my-dynamo-table"

で、その書き換えたテンプレートをAWSコンソールでデプロイする。

