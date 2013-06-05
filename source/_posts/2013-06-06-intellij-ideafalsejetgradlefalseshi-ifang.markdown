---
layout: post
title: "IntelliJ IDEAのJetGradleの使い方"
date: 2013-06-06 02:55
comments: true
categories: IntelliJ, groovy
---

<a href="http://www.jetbrains.com/idea/features/javascript.html" style="display:block; background:#0d3a9e url(http://www.jetbrains.com/idea/opensource/img/all/banners/idea468x60_blue.gif) no-repeat 0 7px; border:solid 1px #0d3a9e; margin:0;padding:0;text-decoration:none;text-indent:0;letter-spacing:-0.001em; width:466px; height:58px" alt="Java IDE with advanced HTML/CSS/JS editor for hardcore web-developers" title="Java IDE with advanced HTML/CSS/JS editor for hardcore web-developers"><span style="margin: 5px 0 0 61px;padding: 0;float: left;font-size: 12px;cursor:pointer;  background-image:none;border:0;color: #acc4f9; font-family: trebuchet ms,arial,sans-serif;font-weight: normal;text-align:left;">Can’t live without</span><span style="margin:0 0 0 205px;padding:18px 0 2px 0; line-height:13px;font-size:11px;cursor:pointer;  background-image:none;border:0;display:block; width:255px; color: #acc4f9; font-family: trebuchet ms,arial,sans-serif;font-weight: normal;text-align:left;">Smart Java and Groovy IDE<br/> with powerfull refactoring and code assistance.</span></a>


こんにちわ、みけです。

いや、もう表題の件に関しては、

いろんなブログで既出なんですけど、
(と思ってググったら、あまりなかった(´・ω・｀))

僕もよくわかってなかったので、

使ってみたときのメモを書いとくことにしました。

JetGradleってそもそも何よ？
---

IntelliJ IDEAに最初からバンドルされているgradleのサポートプラグインです。

では使い方を見て行きましょう。

### (1)JetGradleタブを開きます。

<img src="//googledrive.com/host/0B4hhdHWLP7RRdHRGZ3ZrZU90Q00" style="width : 420px;" />

画面右下の方にちょこんとある、**JetGradle**タブをクリックして開きます。

<img src="//googledrive.com/host/0B4hhdHWLP7RRMEw3b0I1OEo0azA" style="width : 420px;"/>


### (2) Addアンカーをクリックして、build.gradleファイルを割り当てます。

ダイアログが出るので、今のプロジェクトのgradleファイルを割り当てます。

<img src="//googledrive.com/host/0B4hhdHWLP7RRS0xDejBZWmo4ekk" style="width : 280px;" />

build.gradleファイルを選択するとこんなかんじになります。

<img src="//googledrive.com/host/0B4hhdHWLP7RRS1FjS0VhU3BKMTQ" style="width : 420px;" />

### (3) Taskタブをクリックします。

残念ながらこの段階ではタスク一覧が表示されません。

<img src="//googledrive.com/host/0B4hhdHWLP7RRN0xMWnpkNVFOS1E" style="width : 420px;" />

### (4) 更新ボタンをクリックします。

先ほどのタブの左上の方にある更新ボタンをクリックします。

<img src="//googledrive.com/host/0B4hhdHWLP7RRZ051S3NqcEJXQkU" style="width : 420px;" />

するとIntelliJがgradle┏( ^o^)┓ﾄﾞｺﾄﾞｺﾄﾞｺﾄﾞｺ┗( ^o^)┛ﾄﾞｺﾄﾞｺﾄﾞｺﾄﾞｺと聞いてきます。
(AA はテキトー)

<img src="//googledrive.com/host/0B4hhdHWLP7RRcWI4dXkwTENHam8" style="width : 420px;" />

右下の方にある、`Gradle settings`アンカーをクリックします。

### (5) GRADLE_HOMEを設定します。

こんなダイアログが表示されます。

<img src="//googledrive.com/host/0B4hhdHWLP7RRVHpuOTl5S09XUFk" style="width : 280px;"/>

僕はgradleのインストールをgvmでやっているので、

どこに`GRADLE_HOME`があるのかよくわからんので、
(覚えろよjk)

とりあえず、ターミナルで確認します。

<img src="//googledrive.com/host/0B4hhdHWLP7RRcVgtcU5JMmo3NlU" style="width : 280px;" />

`GRADLE_HOME`の値をコピペします。

<img src="//googledrive.com/host/0B4hhdHWLP7RRdFV5cDB0d2IyUzA" style="width : 280px;" />

OKボタンをクリックすると、IntelliJがgradleファイルの読み込みを始めます。

<img src="//googledrive.com/host/0B4hhdHWLP7RRY19neTRiY2Njd2M" style="width : 450px;" />

読み込みが完了すると、タスク一覧が表示されます。

<img src="//googledrive.com/host/0B4hhdHWLP7RRcU9JTnpHanlHNGM" style="width : 450px;" />

### (6)JetGradleからタスクを実行してみる

とりあえず、テストを実行してみたいと思います。

タスク一覧から実行したいタスクを選択します。

<img src="//googledrive.com/host/0B4hhdHWLP7RRbGNhODNIOWFBd1U" style="width : 450px;" /> 

ここでは`test`タスクを実行したいので、

`test`タスクを選択しています。

選択したタスクを長押し(Windowsの場合は右クリック)して、

実行する方法(デバッグとかrunとか)を選択します。

ここではRunを選択します。

<img src="//googledrive.com/host/0B4hhdHWLP7RRUFhIa2RhVDF0QlE" style="width : 450px;" />

テストが実行されました。

<img src="//googledrive.com/host/0B4hhdHWLP7RRT0JoWjlKZXRyRWM" style="width : 450px;" />

これで、一々IDEからターミナルに移動して

```
$ gradle test
```

とか打たなくてもいいですね。

### (7)JetGradleの成果物はどこにあるの？

これ、疑問に思ったので、

あえてテストをこけさせてみました。
(こうすることでテストレポートの位置を示してくれる)
 
わざと落ちるテストを書いて、

Recent Taskからtestをじっこうします。

<img src="//googledrive.com/host/0B4hhdHWLP7RRTGJOeEtoWDJBODQ" style="width : 450px;"/>

はい、落ちました。

<img src="//googledrive.com/host/0B4hhdHWLP7RRTnBXcndheThEV1k" style="width : 450px;" />

で、テストレポートの場所を見ると、

ターミナルからテストを実行した時と変わらない場所に出力されていることがわかります。


以上、結論

### JetGradle便利すなー


 
{% render_partial _includes/post/post_footer.html %}

