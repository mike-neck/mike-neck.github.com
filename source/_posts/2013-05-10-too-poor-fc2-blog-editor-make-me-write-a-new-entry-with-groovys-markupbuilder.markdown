---
layout: post
title: "FC2ブログのエディターがクソいのでGroovyのMarkupBuilderで記事を書いた件"
date: 2013-05-10 20:00
comments: false
categories: groovy
---

こんにちわ、自虐的なみけです。

自虐的なツイートをtogetterにまとめたら、

**自虐的だからたかが知れている、ビジネスなめてんの**

とコメントいただきました。

はい。

### FC2ブログのエディターがクソな件

まあ、このブログを始めると同時に、

[ドラクエ用のブログをFC2](http://mikeneckdq.blog.fc2.com/)の方に

書いているわけですが、

FC2のあのブログエディターなめてんの？

テーブル書いたら

わけわからん改行が出てくるし、

スタイルタグはシカトされるし、

UI舐めきってて、ビジネスを舐めているとしか考えられん。

### しかたがないのでタグの中にstyle属性を

書くことにしたわけですが、

テーブルの要素一つ一つにstyleを手でちまちま書いていた日には、

いくら暇すぎる僕でも、

忙しくなりすぎる。

しかたないので、

**Groovy** の **groovy.xml.MarkupBuilder** を

使うことにしました。

{% gist 5551749 %}


### 何をやろうとしてたかというと、

仕訳を作ろうとしていたわけで、

いくらの投資でいくら回収できるという

話をするのに複式簿記がよくて、

それをGroovyで自動集計させながら、

最終的に総勘定元帳を作り出すというもの。


**MarkupBuilder便利すな**


### ところで…

このスクリプトを書いている時に、

+ 簿記 (Bookkeeping)
+ 仕訳 (Journal)
+ 借方 (Debit)
+ 貸方 (Credit)

といった基本的な単語がわかってないことが

わかりました。

<blockquote class="twitter-tweet" lang="ja"><p>簿記３級くらいの知識が欲しい</p>&mdash; 器官なき身体さん (@mike_neck) <a href="https://twitter.com/mike_neck/status/332655391110467585">2013年5月10日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


というわけで、

簿記・会計のドメインに強くなるべく、

<iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=0071779752" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>

この本を買ったとさ。

おわり。


{% render_partial _includes/post/post_footer.html %}


