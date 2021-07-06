+++
title = "情報共有の方法"
date = "2021-04-26"
draft = true
tags = ["Systems Manager", "Secret Manager", "CloudMap", "CloudFormation"]
+++

* 複数のCloudFormationスタックで同じのリソースを共有する場合にどうするか
* CloudFormationスタックではないシステムとリソースを共有するにはどうするか

こうしたシステム間で共有するパラメーターの保管には

* Systems ManagerのParameter Store
* Secret Manager
* CloudMap
* CloudFormationのOutput

が候補として考えられるが、どれも一長一短という印象がある。

CDKをメインで使う場合に今（2021-04-26）の時点で一番使いにくいのがCloudMapだと思う。
CloudMapは最も汎用的に使えそうなサービスなんだけど、とにかくCDKでデータを読み込んだりするのに向かない。

CloudFormationのOutputは、スタック間で情報を共有するには良さそうだ。
アウトプットしたパラメーターをインポートしているスタックを見つけることや、
インポートされている場合スタックを削除する前に警告がされるという仕組みもある。

ただCloudFormationスタックではないシステムから値を参照するにはAPIコードなどをある程度繰り返さないとデータが取れない