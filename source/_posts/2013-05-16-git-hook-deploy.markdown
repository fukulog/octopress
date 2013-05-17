---
layout: post
title: "git hooks を利用したデプロイを導入しました"
date: 2013-05-16 20:14
comments: true
categories: 
author: wata_n
---

最近ちょっと開発環境を見直していて、その中の一つとして **git hooks** によるデプロイを導入したので紹介してみたいと思います。  

もともとバージョン管理には Subversion を使っていて、今回 git に移行したタイミングで、
git-hooks デプロイを導入しました。  

svn から git への移行については今回割愛しますが、 `git push origin master` とやると自動的に本番にデプロイする方法は、
個人で借りてるサーバでも利用している方法だったので、是非ともやりたかったことの一つ。 

## git hooks

``` bash
$ tree .git/hooks/
.git/hooks/
├── applypatch-msg.sample
├── commit-msg.sample
├── post-update.sample
├── pre-applypatch.sample
├── pre-commit.sample
├── pre-push.sample
├── pre-rebase.sample
├── prepare-commit-msg.sample
└── update.sample

# 実際に動かすときは
$ mv .git/hooks/post-update.sample .git/hooks/post-update
$ chmod +x .git/hooks/post-update
```

git には hooks という、updateやreceiveした後など、いくつかのタイミングをフックしてスクリプトを実行出来る仕組みがあります。
svn でもフックスクリプトは使えますが、こちらはコミットに対してフックする感じで、 git の方がタイミングが豊富という印象があります。    

これの何が嬉しいかというと、例えば、チームでコード品質やコード規約を守るために、コミットする前にコミットしようとしているコードを必ず validation してNGならコミット中止、
コミット後に diff を通知する、push 受けた際にテストを走らす、などなどをするのに便利です。    

今回は *push があった際に更新しようとしているブランチごとに実行される* `post-update` を使ってデプロイ作業の自動化を行いました。    

## デプロイの仕組み

（あ、Gitlab使ってます…）  

{% img https://cacoo.com/diagrams/5qxrWi2Z5Urv7nB2-A183B.png git-hooksデプロイ %}

今回やりたかったこととして、  

- プッシュしたタイミングでtest、staging、本番、の各サーバに即時にソースを反映させたい
- ブランチ毎にWebサーバを切り替えたい

があります。  

一つ目について、    

今まで svn 時代には、  
①ローカルから中央リポジトリにコミット  
②test、stagingサーバにあるワーキングリポジトリ側で中央リポジトリの変更を受け取る（cron に3分毎に `svn up` するようにセットしていた）  
③stagingサーバに ssh で入ってstagingから手で本番へ反映    

のようなフローでデプロイを行なっていましたが、これは git hooks （`post-update`）を使って自動化することで解決しました。

2つ目について、    

`post-update` はタイミング的に `post-receive` と似ているのですが、複数のブランチへのプッシュがあったときに `post-receive` が実行されるのが一度だけなのに対して、
`post-update` はブランチ単位でそれぞれ一度ずつ実行されるという[特徴があります](http://git-scm.com/book/ja/ch7-3.html)。
そして第一引数にブランチ名を受け取るのですが、ブランチ名は `post-update` の中でこんな風に取得することが出来ます。


``` bash
#!/bin/bash

# push されたブランチ名が BRANCH に入る
BRANCH=$(git rev-parse --symbolic --abbrev-ref $1)
```

あとは `$BRANCH` 毎に処理の振り分けをします。  

``` bash
case "$BRANCH" in
	"develop" | feature*)
		develop, feature用の処理
		;;
	release* | hotfix*)
		release, hotfix用の処理
		;;
	"master")
		master用の処理
		;;
esac
```

最終的には `rsync + ssh` でWebサーバへ送りたい、ので一度 `$TMPDIR` に checkout してます。  

``` bash
env GIT_WORK_TREE=$TMPDIR git checkout -f $BRANCH
```

checkout したディレクトリに移動して `rsync + ssh` 。

``` bash
cd $TMPDIR
$RSYNC ./ $USER@$HOST:$DESTDIR/
```

この方法だと、それぞれの各確認用サーバでは常に最後に push されたブランチが当該Webサーバに反映されていることになります。    


アイデア次第でまだまだ面倒な作業を自動化出来そうですが、まだ git による運用自体に慣れてないこともあり、しばらくこれでまわしてみて、
今後運用の中で、また何か改善点など出てきたら共有していきたいと思います。

## tips

tips というかGitlabを使っていてハマった点なのですが、post-updateスクリプトで tmpdir に git checkout したあと、
勝手にディレクトリが`770`、ファイルが`660`というパーミッションになってしまって困った、ということがありました。
post-updateに書いてある内容を手で実行してみてもそんなことにはならず、なんだ？という感じだったのですが、
`home/git/.gitolite.rc`にこんな記述があり、

``` bash .gitolite.rc
%RC = (
    # if you're using mirroring, you need a hostname.  This is *one* simple
    # word, not a full domain name.  See documentation if in doubt
    # HOSTNAME                  =>  'darkstar',
    UMASK                       =>  0007,
```

ユーザがログイン後に実行されるコマンド`/home/git/gitolite/src/gitolite-shell`内で、

``` bash gitolite-shell
# call this once you are sure arg-1 is the username and SSH_ORIGINAL_COMMAND
# has been setup (even if it's not actually coming via ssh).
sub main {
    my $id = shift;

    umask $rc{UMASK};
```

のように指定されているのでした。なので`UMASK => 0002`にしてディレクトリが`775`、ファイルが`664`となるようにして対応しました。  


...と書いたのですが、Gitlab5.0からはgitoliteは[使用しない](http://blog.gitlab.org/gitlab-without-gitolite/)ようなので、
気にしなくていいかもしれません。ぐぬぬ。
