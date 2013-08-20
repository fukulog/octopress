---
layout: post
title: "Google Apps Script を使って各種有効期限を管理＆メールでお知らせするようにすると便利"
date: 2013-08-20 18:55
comments: true
categories: 
author: wata_n
---

GAS（Google Apps Script）を使って、ドメインやSSL証明書などの有効期限を管理＆メールでお知らせするようにしたら便利だった件。

更新が必要系サービスで、有効期限直前に慌てて更新手続きすることってありますよね。
ドメインだったり、Apple Developers Program だったり、iOSアプリに必要な証明書だったり。

それらをちゃんと管理しようと思って、最初はGoogleカレンダーでいいじゃん、って思ったんですが、
1画面で利用サービスを一覧出来る感じも欲しいなぁ…と。リマインドしてくれるだけならGoogleカレンダーで十分なんですけどね。

というわけでGAS！便利です！GASって言っても、JavaScriptだし！

## 有効期限管理表をGoogleスプレッドシートで作る

まずは、こんな表をGoogleスプレッドシートで作ります。

{% img /images/gas_sendmail.jpg Googleスプレッドシート %}

もちろんカラムやカラム名などご自由に。



## スクリプトを書く

ここから実際にスクリプトを書いていきます。
上で作ったスプレッドシートの画面で、[ツール]→[スクリプト エディタ]を開きます。
別画面でエディタ画面が立ち上がると思うので、カラム名やメールなどを埋めつつ、下のようなスクリプトを書いていきます。

``` js
function sendMailExpiredServices(){
  /*
   * Config
   */
  // カラム名
  var SERVICE_COL_NAME = 'サービス';
  var LIMIT_COL_NAME   = '期限';
  var ADMIN_COL_NAME   = '管理者';
  var COMMENT_COL_NAME = 'コメント';
  
  // メール
  var admin = 'nagasawa@gmail.com'; // 管理者（必須）
  var to    = 'nagasawa@gmail.com, foo@example.com, bar@example.com';
  var opts  = {
    //cc   : '',
    bcc  : admin,
    reply: admin
  };

  // 何日前に送信するか
  var grace_time = 1000 * 60 * 60 * 24 * 7;

  /*
   * Main
   */  
  try {
    var sheet = SpreadsheetApp.getActiveSheet();
    var range = sheet.getDataRange();
    var today = new Date();

    for (var i = 2; i <= sheet.getLastRow(); i++) {
      var subject = "【期限切れ】";
      var body = "の期限が迫っています。\n\n"
      + "------------------------------------------------------------\n";
      var footer = "------------------------------------------------------------\n\n"
      + "更新の手続きをお願いします。\n\n"
      + "※更新後はシートの【期限】も更新してください\n"
      + "<Googleスプレッドシートの共有リンク>\n";
      var flg = false;

      for (var j = 1; j <= sheet.getLastColumn(); j++) {
        var col_name  = range.getCell(1, j).getValue(); // カラム名
        var col_value = range.getCell(i, j).getValue(); // 入力値

        body += "【" + col_name + "】\n";
        body += col_value + "\n\n";

        if ( col_name === SERVICE_COL_NAME ) {
          subject += col_value;
          body = col_value + "\n\n" + body;
        }
        if ( col_name === LIMIT_COL_NAME ) {
          var diff = col_value - today;
          if (diff < grace_time) {
            range.getCell(i, j).setBackgroundColor('#fAA');          
            flg = true;
          }
        }
      }
      body += footer;
      
      if (flg) {
        MailApp.sendEmail(to, subject, body, opts);
      }
    }
  } catch(e) {
    MailApp.sendEmail(admin, "【失敗】有効期限管理表からメール送信中にエラーが発生", e.message);
  }  
}
```

スクリプトを保存（フロッピーボタン）して、試しに実行（再生ボタンみたいなやつ）すると、
スプレッドシートの方で、期限が近づいてるセルが赤くなったと思います。



## 届いたメール

そして実際に届くメールがこんな感じ。

``` plain
Subject: 【期限切れ】iOS Development

iOS Development

の期限が迫っています。

------------------------------------------------------
【サービス】
iOS Development

【期限】
Tue Aug 20 2013 00:00:00 GMT+0900 (JST)

【管理者】
nagasawa

【コメント】
開発用プロビジョニングファイル

------------------------------------------------------

更新の手続きをお願いします。

※更新後はシートの【期限】も更新してください
https://docs.google.com/...

```

あとは、スクリプトエディタ画面中、[リソース]→[現在のプロジェクトのトリガー...]を開いて、
スクリプトを実行するトリガーとなるイベントの設定をしときます。

[sendMailExpiredServices] [時間主導型] [日タイマー] [午前9時〜10時]

みたいな感じに。

あとは管理したいサービスが増えてもこのシートにどんどん追加していけばおｋ！

これでもう有効期限直前に慌てるなんてこともないはず！
（あとは人間の問題。。）
