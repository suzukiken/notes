+++
title = "リソースの共有について"
date = "2021-05-02"
tags = ["CDK"]
draft = true
+++

CDKでシステムを作る時に必要なリソースを分けて作る。これは例えばDjangoとMySQLのデプロイは別に行うということと似たような話だけど、自分がCDKで何かを作るという場合はもう少し細かい分け方をする。例えば認証基盤となるCognitoの設定とそれを利用するサービスは別のCloudFormationスタックにするしCDKのプロジェクトも別にしてGitリポジトリも別にしている。

単に自分がそうしているというだけだけど、普通に作ったらそうなるよね、と思う。

さらにこうしたことは認証基盤とかデータベースとかだけではなくて結構たくさんあって、例えば1つのS3のバケットを複数のサービスで共有するとか、SNSトピック然りGraphQLのエンドポイント然りと、やたらいっぱい出てくると思う。

その時にどうやってそのバケットやトピックや認証基盤の情報を共有するのかについては、前に[情報共有の方法](/aws/parameter-sharing)に書いたようにいくつかの方法があると思うけれど、個人的な試作の範囲では99%ぐらいはCDKで何かを作っているし今後もそのつもりだからこそなのかもしれないけど[CloudFormation OutputsとImportValue](/aws/cdkimportexport-context)を使っている。

これはもちろんサービス停止を時々行うことができる前提だからなのだが、まあ最終的に自分が作りたいシステムというのもせいぜいその程度のものだったりするので、すごく後悔するまではこれでいいかという風に思っている。

で、ま、ともかくそのOutputとImportをメインで使っていくつかのCDKプロジェクトを連携させたものを作ってみた。4つのリポジトリに分かれていて大体こんな風になっている。

* [Cognitoの設定をするスタックのリポジトリ](https://github.com/suzukiken/auth1)
  * UserPoolやIdPoolのid諸々はAmplifyのJavaScriptで利用するのでOutputしておく
  * IdPoolの認証ロールもGraphQLの呼び出し権限後述のCodePipelineがつけるのでOutputしておく
* [AppSyncのAPIを作るスタックのリポジトリ](https://github.com/suzukiken/miscapi)
  * GraphQLエンドポイントはAmplifyで利用するのでOutputしておく
  * GraphQLのARNは後述のCodePipelineが認証設定に利用するのでOutputしておく
* [CodePipelineでのデプロイのためのスタックのリポジトリ](https://github.com/suzukiken/webdeploy)
  * ImportしたGraphQLを呼び出す権限をIdPoolの認証ロールに付与する
  * AmplifyのJavaScriptに必要なパラメータをImportしてデプロイ時の環境変数に設定する
* [Amplify（ウェブUI）のリポジトリ](https://github.com/suzukiken/webui)
  * buildspecで`envsubst`を使って環境変数をJavaScriptファイルに反映させるようにしておく