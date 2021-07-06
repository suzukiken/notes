+++
title = "Shopifyアプリ認証フローをLambdaで作る"
tags = ["Shopify", "Lambda", "ApiGateway", "Route53", "S3"]
date = "2021-05-02"
+++

AWS EventBridge経由でさまざまな通知をShopifyから受け取ることができる。
[参考](https://shopify.dev/tutorials/manage-webhook-events-with-eventbridge#requirements)。

なのだがそれはPrivateアプリではダメでPublicかCustomアプリである必要がある。

ShopifyのアプリのうちPublicとCustomはOAuthでの認証が必要となる。
[参考](https://shopify.dev/tutorials/authenticate-with-oauth)

これの仕組みが案外面倒でCustomアプリなら[ローカルでngrokを使う方法](https://www.shopify.com/partners/blog/shopify-admin-authenticate-app)で手軽に認証できるのかもしれないけど、自分の場合はCloud9を使っているので書いてある通りにはできない。

なのでちょうどゴールデンウィークで雨だし、普通に[Authenticate with OAuth](https://shopify.dev/tutorials/authenticate-with-oauth)の通りの仕組みをcdkで作ってみた。

[Githubのリポジトリ](https://github.com/suzukiken/cdkshopifyouth)

かなり横着しているので本番で使う場合には所々直すことにはなるけど、アプリの登録までは普通にできる状態にはなっている。

ちなみに[Authenticate with OAuth](https://shopify.dev/tutorials/authenticate-with-oauth)のページの内容の中でもう少し詳しく書いてあったら助かるんだよなあと思った箇所は以下の部分だ。

* "hush"と書いてあるがそれそのままではダメで、その部分にはAPIシークレットキーを設定する必要がある。
* scopesのカンマをURLエンコードする必要はない。
* 指定したscopesによっては認証が成功しないように思う（自分の勉強不足なだけで理由があると思う）。
* redirect_uriはURLエンコードする必要がある。

ともかく動いてしまえばとても快適だ。cdkだからいくらでも量産できるしね。

ところで今回は気がつかなくて使わなかったんだけど、もしかして[Shopify/shopify_python_api](https://github.com/Shopify/shopify_python_api)を使うと楽だったりするのかもしれない。

なおShopifyはパートナーや開発ショップなどどこからログインするのかわけがわからなくなってしまっていたのだが、とりあえずこのページから進んで行けば良いみたいだ。初めて気がついたけど前からあったんだっけ？[https://accounts.shopify.com/](https://accounts.shopify.com/)
