---
layout: post
title: "FUKULOGのシステム構成・国内IT系各社の開発言語"
date: 2012-06-06 10:49
comments: true
categories: 
---

こんにちは、FUKULOGのnagasawaです。
ここで軽くFUKULOGのシステム構成について書いてみたいと思います。

## OS

CentOS（Amazon EC2）を使用しています。

## Webサーバ

Apache2.2です。EC2の[Elastic Load Balancing](http://aws.amazon.com/jp/elasticloadbalancing/ "Elastic Load Balancing")を利用してロードバランシングをしています。

## アプリケーション

フレームワークなどは特に使用せず、PHPで独自に実装しています。テンプレートエンジンにSmartyを使用しているぐらいです。

## DB

MySQL 5.0 で、全文検索Sennaが組み込まれた[Tritonn](http://qwik.jp/tritonn/ "Tritonn")を使用して、コーディネート/アイテムなどに付随する情報の全文検索を行っています。

また一部では、全文検索エンジンライブラリLuceneがベースになっている[Solr](http://lucene.apache.org/solr/ "Solr")を使った全文検索も行なっています。
Tritonn使ってるのになんで？という感じですが、Tritonnだけでは思うようなパフォーマンスが出なかった、という経緯があります。
ただし最近になって、Tritonn本体とは別の原因がありそう...という話も出ていて一度ちゃんと調査してみる予定です。

## 開発言語

PHPです。
ちなみにFUKULOGではブログパーツの提供もしているのですが、こちらはJava（Google App Engine）で実装されていたりします。

と大まかに書いてみましたが、せっかくなので以下に主に国内IT系各社の主要開発言語を調べて（採用情報や開発者ブログ等から）列挙してみました。

※会社単位で一概に開発言語"これ"とも言えないと思うので参考程度に

- - -

### Ruby

* Cookpad
* Increments（Qiita）
* VASILY（iQON）
* Twitter（Scala、Javaへ移行？）

### PHP

* Yahoo! Japan（米Yahoo!も？）
* 楽天
* GREE
* KLab
* ドワンゴ
* pixiv
* nanapi
* Facebook

### Perl

* はてな
* NHN（旧ライブドア）
* mixi
* DeNA
* KAYAC
* FreakOut

- - -

思いつくまま挙げてみたんですけど、意外にPerl率高めというところでしょうか。
最近のスタートアップに限定してみると、また違う結果になりそうです。


世界的にみると、Pythonが食い込んできたり、Rubyがもう少し多かったりしそうですね。
上場で話題のFacebookでは既存のPHPを独自に高速化（C++に変換）して使ってる、という話も聞いたことがあります。


開発言語は、普通作る"もの"によって最初に決めると思いますが、これから新たに作り始めるとしたら、Ruby on Railsの登場で開発コストが下がったという面で、Rubyが選ばれることが多いんでしょうか。


この際FUKULOGもRubyで...！？（笑）。