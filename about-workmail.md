+++
title = "SESからWorkMailを経由してGmailに転送する"
date = "2021-02-28"
tags = ["SES", "WorkMail"]
+++

SESでメールを受信して、それをGmailに転送する、というのができないかと思ったのだけど、デフォルトのSESのルールだけではできない。

もしそういうことをやるなら、SES、Lambda、S3を組み合わせれば可能であるらしい。
[AWS Blogの記事](https://aws.amazon.com/jp/blogs/messaging-and-targeting/forward-incoming-email-to-an-external-destination/)

でもこれは結構面倒でやる気にならない。

そこで別途費用はかかるけど簡単にやる方法としてWorkMailを使う方法がある。

WorkMailは1アカウント月額4USDかかってしまうけれど、Gmailにメールのリダイレクトや転送をすることができる。

WorkMailのアカウントを作るとまずSES -> WorkMailのルールが自動的に作成されるので、あとはWorkMailのルールとして、メールが来たらGmailに転送するように設定すれば良い。

WorkMailでメールの処理ルールを設定には「転送」と「リダイレクト」があって、転送はいわゆる「FW」がタイトルにつくような受け取り方になり、リダイレクトはそうした変更はされず、あたかもGmail側でメールの受信をしているような形になる。

なお、この機能はGmail専用なわけではなくて、それ以外のメールアドレスも指定することができる。

設定のやり方：

![img](/img/2021/03/workmail-redirect.gif)