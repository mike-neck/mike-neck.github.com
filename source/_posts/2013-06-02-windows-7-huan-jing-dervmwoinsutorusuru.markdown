---
layout: post
title: "Windows(7)環境でgvmをインストールする"
date: 2013-06-02 15:28
comments: true
categories: groovy
---

<img src="https://googledrive.com/host/0B4hhdHWLP7RRZV9ueElsQUlYLVU" style="width : 500px"/>


こんにちわ、みけです。

Windowsでのgroovyの環境構築といえば、

zipを落としてきて、解凍して、パスを通してといった作業をしなければならないのですが、

最近それが結構面倒くさいなと感じて、

Windowsから疎遠になっていました。


gvmをインストールすればいいんですが、

gvmのインストールコマンド

```
curl -s get.gvmtool.net | bash
```

はい、Windowsではできませんね。


#### cygwin

WindowsでGNUな環境を構築するといえば、cygwinですが、

僕は自分が認める(他人はどうだかよく知らん)情弱なので、

cygwin環境の構築に挫折した人です。

で、cygwinでgvmインストールできるよと言われても、

その元になるcygwinが構築できないので、

Windowsでgvm使えないじゃんと思っていました。

#### mingw

Windowsにはいっているmsysgitをいじっていて、

mingwなら構築できそうだと気づいたので、

やってみたら意外とできたので、

その作業メモを書いておきます。


ちなみに、参考にしたページはこちらです。

+ [最近のWindowsの開発環境のセットアップ - 純粋関数空間](http://tanakh.jp/posts/2013-05-23-windows-setup.html)


(1)Chocolateyのインストール
---

Windowsのパッケージ管理ツールです。

[http://chocolatey.org/](http://chocolatey.org/)に書いてある

スクリプトをコピペしてインストールします。


(2)mingwのインストール
---

次のコマンドでインストール出来ます。

```
C:\> cinst mingw
```

なお、インストールの際に

+ C++ Compiler
+ MSYS Basic System
+ MinGW Developer Toolkit

も選択しておいたほうがいいのかなと思って、選択しています。
(よくわかってない…)


(3)msys-unzipのインストール
---

ここからはmingwでの作業になります。

なお、先ほどのリンクでも紹介されている[ConEmu](http://chocolatey.org/packages/ConEmu/12.4.17.1)で、Taskの設定で

```
C:¥MinGW¥msys¥1.0¥bin¥bash.exe --login -i
```

で、mingwを起動できるようにして、

そのコンソールから使っています。

```
mingw-get install msys-unzip
```

これでインストール出来ます。


(4)gvmのインストール
---

ここまでくれば

```
curl -s get.gvmtool.net | bash
```

を実行することができます。


これで、gvmでらくらくgroovyを扱えますね。

<img src="https://googledrive.com/host/0B4hhdHWLP7RRMXctLWN4TmNQdkU" style="width : 408px; height : 280px;">


{% render_partial _includes/post/post_footer.html %}

