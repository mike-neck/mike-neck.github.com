---
layout: post
title: "SourceTreeで細かくコミットする"
date: 2013-05-18 12:58
comments: true
categories: git
---

gitのコミットの粒度はなるべく細かいほうがいいですよね
====

こんにちわ、みけです。

今日はgitの話です。

表題にあるようにgitは細かくコミットしたほうがよいです。

なぜなら、

+ mergeが簡単にできる
+ やらかした時に細かいレベルで元に戻れる
+ 細かければpushするときにまとめればいいや的な

といったメリットがあるからと思っています。

詳細はこっちの記事読んだほうがいいです。

+ [gitのcommitは軽く考えとく - 日々常々](http://d.hatena.ne.jp/irof/20120905/p1)
+ [「git commit するまえに考えるべき10のこと」がDVCS的じゃない件 - うさぎ組](http://d.hatena.ne.jp/kyon_mm/20120905/1346829171)
+ [コミットの粒度 - プログラマの思索](http://forza.cocolog-nifty.com/blog/2012/09/post-fdab.html)
+ [git flow でのチーム開発ワークショップ資料 - Yamashiro0217の日記](http://d.hatena.ne.jp/Yamashiro0217/20120903/1346640190)
+ [git commit するまえに考えるべき10のこと - Act as Professional](http://hiroki.jp/2012/09/05/5523/)


### いや、そんなこと言ったってコミット頻繁にしてないっすよ

これ、僕のことで、そもそもコミットを頻繁にしていないので、Work Treeに変更がたくさん溜まってしまいます。

そうすると、僕のようなdullな人が`git add`する時は`git add -p`で小分けにしながらステージに載せていくのですが、Hunkのサイズを変えるのに一々`git add -p`の後に`s`をしているとわりと面倒い。([参考 - Can I modify git-add's hunk size?](http://stackoverflow.com/questions/1122210/can-i-modify-git-adds-hunk-size))

まあ、かといって、「gitできない奴はVSSでもやってろよ」とか言われると辛い…


### そこでSourceTreeですよ

Atlassianの回し者ではありませんが、SourceTreeがそんな僕には便利です。

#### 変更が溜まりに溜まったwork tree

[{% img https://googledrive.com/host/0B4hhdHWLP7RRZ2hCam1Wa3BLVjA 313 234 %}](https://googledrive.com/host/0B4hhdHWLP7RRZ2hCam1Wa3BLVjA)


#### hunkがでかすぎるので小さくします。

[{% img https://googledrive.com/host/0B4hhdHWLP7RRdEN5VTRHVE5hNVU 364 304 %}](https://googledrive.com/host/0B4hhdHWLP7RRdEN5VTRHVE5hNVU)


#### hunkが細かくなります。

[{% img https://googledrive.com/host/0B4hhdHWLP7RRanRRZjA2VVh6eXc 362 270 %}](https://googledrive.com/host/0B4hhdHWLP7RRanRRZjA2VVh6eXc)


#### さらに行単位で変更を選びます

[{% img https://googledrive.com/host/0B4hhdHWLP7RRdTVodnZ6MDF1c1U 364 68 %}](https://googledrive.com/host/0B4hhdHWLP7RRdTVodnZ6MDF1c1U)


#### 意味のある単位にまとまったのでコミットします

[{% img https://googledrive.com/host/0B4hhdHWLP7RRUEQ3bnkxTTVzQ0E 412 234 %}](https://googledrive.com/host/0B4hhdHWLP7RRUEQ3bnkxTTVzQ0E)

commitボタンを推します。

[{% img https://googledrive.com/host/0B4hhdHWLP7RRc1ZUQlV3akplUFU 333 270 %}](https://googledrive.com/host/0B4hhdHWLP7RRc1ZUQlV3akplUFU)

#### 適度な粒度でコミットしたのでプッシュします。

[{% img https://googledrive.com/host/0B4hhdHWLP7RRcHI5NGt0aksyZVk 310 175 %}](https://googledrive.com/host/0B4hhdHWLP7RRcHI5NGt0aksyZVk)

pushボタンのところにプッシュされていないコミットの数が出ています。

pushボタンを押してプッシュします。

[{% img https://googledrive.com/host/0B4hhdHWLP7RRdEJ2aGhZeGM2LVU 446 180 %}](https://googledrive.com/host/0B4hhdHWLP7RRdEJ2aGhZeGM2LVU)

プッシュ完了すると、ローカルとリモートが一致した状態になります。

[{% img https://googledrive.com/host/0B4hhdHWLP7RRd18wb2JRNWxFYkU 471 90 %}](https://googledrive.com/host/0B4hhdHWLP7RRd18wb2JRNWxFYkU)


### 細かい粒度でコミット出来ましたね

+ SourceTree、マジ便利、マジ神、ハンパないです。
+ いや、そもそもそんなにWork Treeに変更を溜め込むなという話ですね…


<table>
<tr>
<td style="padding : 2px;"><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=4798023809" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
<td style="padding : 2px;"><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=4798035009" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
<td style="padding : 2px;"><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=477415184X" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
</tr>
<tr>
<td style="padding : 2px;"><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=4274068641" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
<td style="padding : 2px;"><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=4873114403" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
<td style="padding : 2px;"><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=427406767X" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
</tr>
</table>


{% render_partial _includes/post/post_footer.html %}


<br/>
