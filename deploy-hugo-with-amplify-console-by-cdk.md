+++
title = "CDKでHugoをAmplifyコンソールでデプロイ"
date = "2021-02-23"
tags = ["Hugo", "Amplify"]
+++

HugoをAmplifyコンソールでデプロイする。

このサイトで今（2021-02-23）やっていることですが、CodeCommitにリポジトリをおいて、Amplifyコンソールでそれを公開するようにしてます。

そうする特別な理由は無いですが、単にAmplifyコンソールはビルドからデプロイまでをまるっとやってくれるので楽だと思っているし、Githubに置くよりCodeCommitの方がAWSとの連携のことを気にしなくて良いだろうと思っています。

## CDKを使う

具体的にはCDKのプロジェクトを一つ用意して、そのプロジェクトの中で以下のものを作っています。

* CodeCommitのリポジトリ
* Amplifyコンソール（カスタムドメインの設定も）

こうして受け皿を用意しておいてから、HugoのプロジェクトをCodeCommitのリポジトリにgit pushするとサイトが更新されるという流れになります。

[CDKのソースコード](https://github.com/suzukiken/cdkamphugo)

ところで疑問なことが1つあって、CDKのコードでbuildSpecを`codebuild.BuildSpec.fromObject`で作るとうまくいかない。

[CDKのリファレンス](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-amplify-readme.html)に書いてあるのをそのまま真似しても動かない。

動かないコード
```TypeScript
buildSpec: codebuild.BuildSpec.fromObject({
    version: '1',
    frontend: {
      phases: {
        build: {
          commands: [
            'wget https://golang.org/dl/go1.15.5.linux-amd64.tar.gz',
            'tar -C /usr/local -xzf go1.15.5.linux-amd64.tar.gz',
            'export PATH=$PATH:/usr/local/go/bin',
            'wget https://github.com/gohugoio/hugo/releases/download/v0.81.0/hugo_extended_0.81.0_Linux-64bit.tar.gz',
            'tar -xf hugo_extended_0.81.0_Linux-64bit.tar.gz hugo',
            'mv hugo /usr/bin/hugo',
            'hugo --baseURL $BASEURL',
          ]
        }
      },
      artifacts: {
        baseDirectory: 'public',
        files: '**/*',
      },
      cache: {
        paths: []
      }
    }
}),
```

これをcdk deployすると、AmplifyコンソールのBuild settingsのところにはamplify.ymlと表示されるけれど、そこに表示されるコードのシンタックスはyamlではなくjsonになってしまうのでそれがダメなのかなと思うんだけど、ビルド自体は成功してサイトは公開されるんですよね。

ただ静的ファイルが表示されないという問題があって結局それの解決方法がわからない。

原因としてもう1つ考えられるのは`files: '**/*'`のところがちゃんとrecursiveな動作をしていないのではないかということだけど[CodeCommitのビルドスペックリファレンス](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)に

> '**/*' represents all files recursively.

と書かれているのでそこが間違っているわけではなさそう。

結局、Amplifyアプリケーション側のリポジトリに以下のようなamplify.ymlを置けばこうした問題は起きないので今はそれでやっています。

```amplify.yml
version: 1
frontend: 
  phases: 
    build: 
      commands: 
        - wget https://golang.org/dl/go1.15.5.linux-amd64.tar.gz
        - tar -C /usr/local -xzf go1.15.5.linux-amd64.tar.gz
        - export PATH=$PATH:/usr/local/go/bin
        - wget https://github.com/gohugoio/hugo/releases/download/v0.81.0/hugo_extended_0.81.0_Linux-64bit.tar.gz
        - tar -xf hugo_extended_0.81.0_Linux-64bit.tar.gz hugo
        - mv hugo /usr/bin/hugo
        - hugo --baseURL $BASEURL
  artifacts: 
    baseDirectory: public
    files: 
      - '**/*'
  cache: 
    paths: []
```