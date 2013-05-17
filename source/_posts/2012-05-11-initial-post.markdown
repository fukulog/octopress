---
layout: post
title: "エンジニアブログはじめました"
date: 2012-05-18 11:00
comments: true
categories: 
---

こんにちは、はじめまして。FUKULOGエンジニアのnagasawaです。
FUKULOGでも開発者ブログを始めることになり、先月入社したばかりのド新人ですが、私が担当することになりました。よろしくお願いします。

初めての記事なので、少し自己紹介を。

最初に触った言語はperlで、今も趣味で何か作ろうとするときにはperlをよく使っています。
FUKULOGの開発言語であるphpの経験は極めて浅く、現在日々勉強中です。。。
業務では主にフロントエンドを担当、既にいくつかの機能の実装をやらせてもらっています。
今後ともよろしくお願いします！

というわけで、開発者ブログを始めるにあたり、FUKULOGの[githubアカウント](https://github.com/fukulog)を作るとともに、個人的にも気になっていた[Octopress](http://octopress.org/docs/)でのブログ運用を導入してみました。
Octopressの特徴として以下のような点が。

* MIT License
* ruby製
* コマンドラインベースでの運用
* 自由度が高いカスタムが可能
* Markdown記法で記事を書く
* 静的にHTMLが作られる（MTに似てる）
* コードの表示が綺麗（→A blogging framework for hackers.）
* githubとの親和性が高い（gistの埋め込みも容易）

``` bash rvmのセットアップ、ruby-1.9.2のインストール、rubygemsの更新
bash < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer)
echo '[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm" # Load RVM function' >> ~/.bash_profile
source .bash_profile
rvm install 1.9.2
rvm use 1.9.2
rvm rubygems latest
```

``` bash Octopressのインストール
cd dev
git clone git://github.com/imathis/octopress.git dev-blog
cd dev-blog
gem install bundler
bundle install
bundle update rake
rake install
```

``` bash ブログのプレビュー
rake generate
rake preview
```

``` bash ブログのデプロイ
rake setup_github_pages
# Github PagesのリポジトリURLを入力
# git@github.com:fukulog/fukulog.github.com.git
rake gen_deploy
```

Octopressでは、WEBrickというrubyのみで書かれたWebサーバー用フレームワークが使われていて、`rake preview`するとWEBrick::HTTPServerが4000番ポートで起動します。
Apacheなどの設定をしなくても手元ですぐにプレビュー出来るのはお手軽で良い感じです。デプロイ時に`rake generate`で自動生成された静的ファイルがgithubにpushされて公開となります。

毎回コマンドラインからプレビューしなくても、
日常、下書きを書きためたり、Markdownをプレビューするときには、[Kobito.app](http://kobitoapp.com/)というMarkdown記法をリアルタイムプレビュー出来るアプリ等を使うのも良さそうかなと思っています。
（※KobitoはMarkdown記法のルールが若干異なるので注意が必要）

まだ手探り状態ですが、しばらくはこれで運用してみたいと思います。

今後もFUKULOGの開発のなかで得た情報を共有・発信していきたいと思うので、よろしくお願いします！
