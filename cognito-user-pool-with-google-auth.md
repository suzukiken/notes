+++
title = "Googleのアカウントを使ってCognitoで認証する 方法2"
date = "2021-02-25"
tags = ["Google", "Cognito"]
+++

組織のGoogle WorkspaceアカウントでCognitoにログイン(SSO)できるようにUser Poolを使う方法。

![img](/img/2021/03/cognitogooglesso2.png)

[前の記事](/aws/introducing-cognito-with-google-auth)に書いたように、SSOのやり方としては２つの方法があって、ここで書くのはそのうちのUser Poolを使う方法です。

## 方法１より複雑

デメリットとしては、CDKのコードが若干長く、設定に必要なパラメータの数が多い。
とはいえCognitoの設定はCDKでできるので、正直あまり面倒ではない気もする。

先のことを考えれば、この方法をとっておく方が良いだろうなあという気はする。
Amplifyでadd authした場合もこの形となる。

## ユーザー管理でできること

* `example.com`以外のユーザーにもシステム利用を許可できる
  * `gmail.com`の特定のアカウントを許可できる
  * Googleアカウントの別のドメインのアカウントを許可できる
* `example.com`の中の特定のメールアドレスを拒否・許可できる
* ID＆パスワードでのログイン機能も追加できる
* ユーザーのグループを作ってグループ毎に異なる権限を付与できる

[Githubのリポジトリ](https://github.com/suzukiken/cdkuserpool)

https://github.com/suzukiken/cdkuserpool/blob/fe6f025f80c27ba41aff00ae1d8e55b323e9b4c1/lib/cdkuserpool-stack.ts#L15-L20
のところにこれらのパラメータを設定する必要がある。

```
GOOGLE_CLIENT_ID = "<xxxx.apps.googleusercontent.com>" // Google APIの認証設定画面から得られる
GOOGLE_CLIENT_SECRET = "<xxx-xxxxxxxxx-xxxxxxxxxx>" // Google APIの認証設定画面から得られる
ALLOWED_EMAILS = "a@example.com, b@example.com," // ログインを許可するGoogleのアカウントのメールアドレス
ALLOWED_DOMAINS = "c.example.com, d.example.com" // ログインを許可するGoogleのアカウントのドメイン
COGNITO_CALLBACK_URL = "https://e.example.com/,https://f.example.com/" // サイトを公開・開発するURL（カンマ区切りで複数可）
COGNITO_LOGIN_URL = COGNITO_CALLBACK_URL // サイトを公開・開発するURL（カンマ区切りで複数可）
```

これらのパラメータをSSMやSMから取り込むようにしたのが[こちらのGithubのリポジトリ](https://github.com/suzukiken/auth1/blob/master/lib/auth1-stack.ts)でパラメータの名前はcdk.jsonから得流ようにしている。

Systems Managerからのデータ取り込みについては[さまざまな関数がある](/aws/cdkssm)けれど、登録された値を変更して`cdk deploy`したときにそれがchange setになってくれるのが望ましかったのでvalueForStringParameterを使った。

またcognito.UserPoolClientを作る際のoauth_settingsに与えるcallbackUrlsやlogoutUrlsはvalueForTypedStringParameter( ..., ParameterType.STRING_LIST)を受け付けないようだったのでcdk.Fn.splitを使っている。

それからcognito.UserPoolIdentityProviderGoogleとcognito.UserPoolClientの間に依存性を持たせないと`cdk deploy`が失敗することがあって、なおこれは成功する場合もよくあるので必須なわけではないがcdk.ConcreteDependableを使って依存させている。

あとcognito.UserPoolClientのoAuthのscopesにcognito.OAuthScope.COGNITO_ADMINを含めないと、どうやらAmplifyのAuth.currentUserInfo()でユーザーのメールアドレスなどを受け取ることができないようだ。