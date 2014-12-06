---
layout: post
title: GitHub Pages への移行
tags: [Ruby, GitHub]
---
brly.net は AngularJS(1.x)とsinatra(ruby) + MongoDBで構成されていて
かなり実装の怪しげなTwitter-OAuthによる認証が組み込まれたCMSだった.

拙作のアプリのため、かなり機能面の不備を感じていて、またセキュリティ的にも
アレだったのでさっさと別のシステムへ移行させようと思い立って
jekyll(GitHub Pages)に移行させた.

#### レポジトリの作成 (brly.github.io)

反映は10分くらいで行われるらしいのだけれど,
とりあえず Hello world 的なものを push して反映するまで30分くらいかかった.

#### チュートリアルを見る

[GitHub using jekyll with pages](https://help.github.com/articles/using-jekyll-with-pages/)

[30分のチュートリアルでjekyllを理解する](http://melborne.github.io/2012/05/13/first-step-of-jekyll/)

とりあえず概要だけ.

#### Pygments のインストール (syntax highlight)

{% highlight bash %}
sudo easy_install pip
sudo pip install Pygments
{% endhighlight %}

そのあとは "30分チュートリアル" の手順にもあるように syntax.css を吐かせて設置する.

#### プラグインを入れる

画像へのパスの記述が簡単になる.

[jekyll-asset-path-plugin](https://github.com/samrayner/jekyll-asset-path-plugin)

その他にもページネーションとか、指定したタグを含むページとか、いろいろやろうと思ったけど疲れたのでパス.

#### その他の作業

- http-equiv, meta タグを他のサイトからパクッてきて使う.
- bootswatch から適当にテーマを選んで使う.

#### TODO

- オレオレフォーマットでMongoDBに保存されている記事の移行
  - 自動化を諦めて適当に pick して人力, ていうのは辛すぎる..
