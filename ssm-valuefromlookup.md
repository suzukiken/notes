+++
title = "Systems MangaerのパラメータをvalueFromLookupで参照することについて"
date = "2021-05-09"
tags = ["Systems Mangaer"]
draft=true
+++

CDKでSystems Mangaerから値を取り出すのにはいくつかの方法がある。[参考](/aws/cdkssm)

いくつかあるメソッドのうちvalueFromLookupはcdk.context.jsonで値の管理をするというものでそこに興味を持った。

## valueFromLookupの動作

まずvalueFromLookupを使うとSystems Managerから取り出した値でcdk.context.jsonが作られその内容でCloudFormationテンプレートが作られる。つまりCloudFormationテンプレートを作る時に内容が確定する。

これはデプロイ時に初めて値が確定する[動的な参照](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/dynamic-references.html)に比べて良し悪しがあると思うけれど、CloudFormationテンプレートに値が書かれている方が問題が起きた時のデバッグはしやすそうな気がする。

もし以前作ったcdk.context.jsonがあればそれを参照してCloudFormationテンプレートが作られるのでSystems Managerの内容は参照しない。

だからSystems Managerの値を変更して`cdk deploy`してもそれはCloudFormationテンプレートには反映されない。なんならそのパラメータが実際にSystems Managerにあるかどうか確認もしない。だからもし誤ってSystems Managerのパラメータを消してしまってもcdk.context.jsonが存在してそこに値が書かれている限りCloudFormationテンプレートの生成はされる。

逆にSystems Managerのパラメータを変更してその新しい値でデプロイしたい場合はcdk.context.jsonを削除してから`cdk deploy`することで、新しい値がSystems Managerから取得されてcdk.context.jsonが新しい値で生成され、その内容でCloudFormationテンプレートが作られる。その結果として[Change sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)が作られるのでそれをデプロイして内容が変更されるという流れになる。

なおcdk.context.jsonを削除してSystems Managerのパラメータも削除した場合は`cdk deploy`するとエラーとなる。

万が一誤ってそういうことになってしまった場合はcdk.context.jsonを自分ででっち上げることはできる。

cdk.context.jsonの中は単純に

```
"ssm:account=<AWSアカウントID>:parameterName=<パラメータ名>:region=<リージョン>": "値",
```

というのが並んでいるだけの階層のないキー・バリューの羅列なのでパラメータ名が明確なら自分でその値をでっち上げることができる。

もっと言えばSystems Managerの内容がどうであるかは関係なくcdk.context.jsonの内容を変更すればそれを元にテンプレートが作られるので自分の使いたい値を直接cdk.context.jsonに書いてしまってもいい。

これは結局CDKのスタックのコードからSystems Managerを参照している部分を削除して代わりに直接値を書いてしまうのと同じことで、どちらの方法でもテンプレートの内容に違いはない。

以上が色々試してわかった動作で大体必要なことの全てであるように思うけどどうなんだろう。個人的には満足した。

### cdk.context.jsonの管理方法

cdk.context.jsonは[AWSのドキュメント](https://docs.aws.amazon.com/cdk/latest/guide/context.html)によればバージョンコントロールに含めることがおすすめされている。要は一緒にGithubに置いとく方がいいよという意味なのかな。

でも自分がなぜSystems Managerを使いたいかという話をすると1つの理由として、Githubでリポジトリのコードをpublicにしつつも必要な時にその自分で作ったコードを使いたい。自分で使う場合のことを考えるとメールアドレスなどがあった場合に、できればpublicなコードに入れておきたくないと思う。

そこでSystems Managerから値を取ってくるようにしたらコードを公開しやすいと思う。

でもvalueFromLookupを使っている場合cdk.context.jsonが生成されてそこに隠したいものが書かれてしまう。これはgitでignoreすればいいわけだけど、だったらそもそもvalueFromLookupを使っている意味があるのだろうか。valueForStringParameterを使えばcdk.context.jsonは生成されない代わりにSystems Managerのパラメータを誤って捨ててしまったりすると面倒ではある。

などいろんなことを思っていて結局どうしているかというと、自分の場合cdk.context.jsonはgitでignoreしつつもS3バケットに放り込んでおくことにした。そのバケットにはバージョン管理も機能させているので何なら過去に遡ることもできる。これが意味があるのかはわからんが。


