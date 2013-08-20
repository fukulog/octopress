---
layout: post
title: "Xcode で コードフォーマッター Uncrustify を使う"
date: 2013-05-22 15:01
comments: true
categories: 
author: wata_n
---

{% blockquote Uncrustify http://uncrustify.sourceforge.net/ %}
Source Code Beautifier for C, C++, C#, ObjectiveC, D, Java, Pawn and VALA
{% endblockquote %}

最近iOS開発で使い始めて、良い感じな Uncrustify 。
設定に従って、インデントの数やスペースの取り方など、自動でコードを綺麗に整形してくれます。
似たもので、 perl 版で同じようなことをしてくれるツール [perltidy](https://metacpan.org/module/SHANCOCK/Perl-Tidy-20121207/bin/perltidy) とかがあります。

新しいソフトウェアとかでは全然無いんですが、Xcode にスクリプトを設定して任意のタイミングで uncrustify を走らす、
というのが便利だった＆インストール〜Xcodeに設定までが一気通貫でまとまった記事があまり無さそうだったので書いてみます。


## インストール

homebrew を使っているならこれだけで入ります。
（ちなみに [Xcode プラグイン版](https://github.com/benoitsan/BBUncrustifyPlugin-Xcode)もありますが、今回は使いません）

``` bash
$ brew install uncrustify
```


## Xcode から実行するスクリプト

スクリプトはホームディレクトリの`~/bin/uncrustify`に、設定ファイルは`~/.uncrustifyconfig`に置いてます。

``` bash
$ vi ~/bin/uncrustify # スクリプト
$ chmod +x ~/bin/uncrustify
$ vi ~/.uncrustifyconfig # 設定ファイル
```

スクリプトは、[Using Uncrustify directly in Xcode 4!](http://robertjpayne.tumblr.com/post/9092159751/using-uncrustify-directly-in-xcode-4) を参考にパスなどを変えてます。

``` ruby
#!/usr/bin/ruby
base_path = ENV['XcodeProjectPath'] + "/.."

puts "running uncrustify for xcode project path: #{base_path}"

if base_path != nil
  paths = `find "#{base_path}" -name "*.m" -o -name "*.h" -o -name "*.mm" -o -name "*.c"`
  paths = paths.collect do |path|
    path.gsub(/(^[^\n]+?)(\n)/, '"\\1"')
  end
  paths = paths.join(" ")
  result = `/usr/local/bin/uncrustify -c ~/.uncrustifyconfig --no-backup #{paths}`;
  puts result
else
  puts "Invalid base path..."
end
```

設定ファイルは [Uncrustifyのセッティング (1) プリプロセッサ編 #uncrustify - Qiita [キータ]](http://qiita.com/items/dd7c5ffdff27451dae16) が詳しかったので、
使える設定、その説明を見ながら書きます。


## Xcode の設定

Xcode で`Build`に成功したタイミングで uncrustify を実行する場合です。
[Preferences…] -> [Behaviors] と進んで、Build の Succeeds を選択し、Run の項目に上で作ったスクリプトを指定します。

{% img /images/xcode_behaviors.png Xcodeの設定 %}

あとは [Product] -> [Build] すると大量に差分が出ます。 差分を見ながら`.uncrustifyconfig`に指定した設定に従って整形されているのが確認出来ると思います。

それぞれコーディング中は好きな作法で書いても、設定ファイルさえチームで共有しとけば、第三者が見て綺麗なソースが常に保てて便利ですね。