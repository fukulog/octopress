---
layout: post
title: "画面スクロールされたかをGoogle Analyticsのイベントトラッキングに記録する"
date: 2013-01-11 10:29
comments: true
categories: 
author: wata_n
---

{% blockquote attachScrollEventTracking https://gist.github.com/4490363 %}
画面のスクロール位置を、Google Analyticsのイベントトラッキングに記録する。 記録したい要素のidを配列として関数に渡すと、その要素が画面に表示されたときに記録を残す。
{% endblockquote %}

つい最近導入してみたんですが、前回につづきjsネタです。

twitterなどからの流入をなんとか活かせないかと、直帰率を下げる＆回遊率を上げるべく、該当ページの改修をすることに。
しかしコンテンツを見直すにも、そもそも何は見られていて、どこで見切られてしまっているのかの情報が無い状態だったので、
こんなjsを書いてGoogle Analyticsで見てみることにしました。

{% codeblock lang:javascript %}
$.extend({
	attachScrollEventTracking: function(eventcategory, pagePartsIdList) {
		var pageParts = {};
		var windowHeight = $(window).height();
		
		$.each(pagePartsIdList, function(key, pagePartsId){
			var target = $("#" + pagePartsId);
			if (target.length > 0) {
				pageParts[pagePartsId] = target.offset().top;
			}
		});
		
		var isFinish = false;
		$(document).bind("scroll", function(event) {
			if (!isFinish) {
				var scrollTop = $(document).scrollTop() + windowHeight;
				
				isFinish = true;
				$.each(pageParts, function(pagePartsId, value){
					if (value < scrollTop) {
						//console.log(['_trackEvent', eventcategory, 'scroll', pagePartsId, value, true]);
						_gaq.push(['_trackEvent', eventcategory, 'scroll', pagePartsId, value, true]);
						delete pageParts[pagePartsId];
					}
					isFinish = false;
				});
			}
		});
	}
});
{% endcodeblock %}

使い方:

{% codeblock lang:javascript %}
$(document).ready(function() {
	// 記録したい要素のidを指定
	var pageParts = ["section-1", "section-2", "section-3"];
	$.attachScrollEventTracking('tracking_event_category', pageParts);
});
{% endcodeblock %}

スクロールイベントに対して、指定した要素の位置までスクロールされたかを判定する処理をバインドし、
Google Analyticsに送信しています。結果はどうかな・・・。

手軽に使える感じなので、今後効果測定やページ解析にも使っていきたいと思います。