---
layout: post
title: "スマホでhover効果を実現するjQueryプラグイン"
date: 2012-12-27 17:45
comments: true
categories: 
author: wata_n
---

{% blockquote jquery.taphover.js https://gist.github.com/4282291 %}
スマホの画面タップ時に要素をhoverさせる CSSは、a:hoverの代わりにa.hoverを使う aタグ以外の場合はclassにtapを指定するとCSSで:hoverの代わりに.hoverを使えるようになる
{% endblockquote %}

PCではリンクやボタンにカーソルが乗っかったときや押したときの挙動として、スタイルシートで
{% codeblock lang:css %}
a:hover { color:red; }
a:active { color:red; }
{% endcodeblock %}
のようにやりますが、スマホサイトではこれが[出来ません](http://oshiete.goo.ne.jp/qa/7480431.html)。

そこでFUKULOGのスマホサイトでもそうですが、

[jQueryを使ってスマホで:hover効果を実現する | アルパカの具](http://minipaca.net/blog/jquery/jquery-de-hover/)
{% codeblock lang:javascript %}
jQuery(function($){
    $( 'a, input[type="button"], input[type="submit"], button' )
      .bind( 'touchstart', function(){
        $( this ).addClass( 'hover' );
    }).bind( 'touchend', function(){
        $( this ).removeClass( 'hover' );
    });
});
{% endcodeblock %}

のようにjsでaタグやボタンを指定して、要素がタッチされたときに`hover`や`active`などのclassを付けることで対応していました。

しかしこれだと、そのページにある無数のaタグやボタンのeventに対して個別にコールバック関数をbindしていくことになり、要素が増えれば増えるほど処理に時間がかかってしまうことになります。
そこでbindするのはwindowのeventに対してのみにし、コールバック関数の中で実際にタッチされている要素を判別するようにしたのが、[jquery.taphover.js](https://gist.github.com/4282291)です。

{% codeblock from jquery.taphover.js %}
$(window).bind("touchstart", function(event) {
	var target = event.target || window.target;

	var bindElement = null;
	if (target.tagName == "A" || $(target).hasClass(tapClass)) {
		bindElement = $(target);
	} else if ($(target).parents("a").length > 0) {
		bindElement = $(target).parents("a");
	} else if ($(target).parents("." + tapClass).length > 0) {
		bindElement = $(target).parents("." + tapClass);
	}

	if (bindElement != null) {
		Hover().touchstartHoverElement(bindElement);
	}
});
{% endcodeblock %}

あらかじめjquery.taphover.jsを全ページで読み込んでおくだけで、あとは必要に応じて、
{% codeblock lang:css %}
article a.hover { color:red; }
{% endcodeblock %}
のような感じで、該当する要素に対して適宜スタイルを追加していく、という使い方が出来ます。

今後スマホサイトで全面的に移行していくつもりです。