+++
title = "Googleのアカウントを使ってCognitoで認証する 方法1"
date = "2021-02-25"
tags = ["Google", "Cognito"]
+++

組織のGoogle WorkspaceアカウントでCognitoにログイン(SSO)できるようにする方法について。

[前の記事](/aws/introducing-cognito-with-google-auth)に書いたように、やり方として２つの方法があるわけだけど、ここで書くのはそのうちのId Poolだけを使う方法です。

![img](/img/2021/03/cognitogooglesso1.png)

## Id Poolだけでできるので手軽

この方法の良いところは

* CDKのコードが短い
* 設定するパラメータ数が少ない
* Google側での設定箇所が少ない

## その代わり細かいことはできない

User Poolを使う場合に比べると、ほとんとユーザー管理らしいことはできません。

つまりユーザー管理などの必要がない場合に選択できる方法ということになる。
具体的には以下の条件に合致する場合にのみ使える方法だ。

* 組織のメンバー全員がサービスにログインできて構わない
* 組織のメンバー全員が同じ権限を持っていて構わない
* 組織外のメンバーは使わない

[Githubのリポジトリ](https://github.com/suzukiken/cdkidpool/)