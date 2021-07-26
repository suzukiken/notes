+++
title = "KicadとFreeRouting"
tags = ["Kicad", "FreeRouting"]
date = "2021-07-25"
+++

Kicadを使っていて自動配線をしようとした場合FreeRoutingを使うことができる。
FreeRoutingはそれ自体が独立したソフトウェアなのでKicadがバージョン5なら、別途インストールする必要がある。

かつてKicad 4には自動配線の機能としてあらかじめFreeroutingが入っていたらしい。
自分もKicad 4を使っていたような気がするが、どんなふうにインストールしたのかはもう覚えていない。
ともかく現時点のKicadにはFreeRoutingは含まれていない。

ここではFreeRoutingをインストールして動作するようにするための手順をメモしておく。
なお、インストールしたマシンはMacBook Air M1 2020モデルで、Mac OSX は11.4 Big Surです。

### FreeRouting関連のリンク

FreeRouting AutoRouter
[FreeRouting | FreeRouting Documentation](https://freerouting.org)

Layout Editorの中に含まれる
[LayoutEditor the universal editor for GDSII, OpenAccess, OASIS… | LayoutEditor](https://layouteditor.com)

あるいはFreeRoutingのGithubのリポジトリからもダウンロードリンクがある
[GitHub - freerouting/freerouting: Advanced PCB autorouter (finally, no Java installation required)](https://github.com/freerouting/freerouting)

### Mac OSX インストール

1. Layout EditorのMac OSX向けのインストーラーでインストールする方法
2. Linux向けのバイナリをダウンロードする

いずれにしても欲しいのはjarファイル。

1の場合、dmgファイルでインストールした後は、目的のjarファイルは`/Applications/layout.app/Contents/MacOS/freeRouting.jar`のようなところに入っているはず。

2の場合、zipを解凍すると`freerouting-1.4.4-linux-x64/lib/app/freerouting-executable.jar`にある。

Java 11が必要なので、それはそれでOracleのサイトからMacOS向けのインストーラ（.dmgファイル）をダウンロードしてインストールする。

### 起動

#### 1の場合
```
java -jar /Applications/layout.app/Contents/MacOS/freeRouting.jar
```

#### 2の場合
```
java -jar freerouting-1.4.4-linux-x64/lib/app/freerouting-executable.jar
```

2の場合はコマンドだけで自動配線が実行されて結果がファイルに書き出される。

例:
```
java -jar freerouting-1.4.4-linux-x64/lib/app/freerouting-executable.jar -de input.dsn -do output.ses -mp 1000
```

例: ルールファイルを用意した場合
```
java -jar freerouting-1.4.4-linux-x64/lib/app/freerouting-executable.jar -de input.dsn -do output.ses -mp 1000 -dr some.rules
```

### 差異

全く同じものというわけではなくて、LayoutEditorに入っている方は、それ用になっているらしい。

![スクリーンショット 2021-07-25 9 24 02](https://user-images.githubusercontent.com/557268/127066472-b0e5e57f-0d24-4ff4-ac57-37c1155073b5.png)

![スクリーンショット 2021-07-25 9 25 32](https://user-images.githubusercontent.com/557268/127066476-359224e8-4a4c-4bb3-8bb2-b52a3c75d39e.png)

### Kicadとの連携方法

連携といってもファイルをお互いにやりとりするだけだし、そのファイルの作成や読み込みなどは、基本的には人間がGUIで行う。

KicadからのPcbnewという基板レイアウトアプリケーションからSpecctra DSNファイルに書き出して、それをFreeRoutingで開いてFreeRoutingに搭載されている自動配線機能で配線を行い、配線が出来上がったらそれをSpecctra Sessionファイルに書き出して、それをPcbnewにインポートするという作業をすることになる。

Kicadにはコマンドがあるようなので、それである程度自動化できるのかもしれないがそちらは未確認。
