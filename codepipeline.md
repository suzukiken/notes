+++
title = "CodePipelineを使って静的サイトをデプロイする"
date = "2021-02-25"
tags = ["CodePipeline", "CodeCommit", "CodeBuild", "CodeDeploy", "Github"]
+++

CodePipelineを使って静的サイトをデプロイする。

![img](/img/2021/03/codepipeline.png)

一度ぐらい使っても大体忘れてしまうのでまとめておくがCodePipelineとは何かというと

* CodeCommitやGithubからのデータ取得
* CodeBuild
* CodeDeploy

の組み合わせでコードをデプロイする仕組みということになると思う。デプロイのフローはこうなる。

1. CodeCommitやGithubからソースコードを得る
2. CodeBuildでビルドする
3. ビルドした内容をS3に置く

基本的な仕組みでは、リポジトリにソースをpushしたことで1がトリガーされてあとは自動で進んでゆく。

[Githubのリポジトリ](https://github.com/suzukiken/cdkcodepipeline)

このサンプルコードは単にS3のバケットにデプロイするだけのものだけどそれをCloudFrontで配信すればいわゆる静的サイトのようになる。[参考](https://github.com/suzukiken/cdkcodepipeline-github-cloudfront)

buildspec.ymlファイルはデプロイされるリポジトリに置いておく必要がある。
