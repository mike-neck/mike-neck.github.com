---
layout: post
title: "AndroidがJavaでない理由 - 1"
date: 2013-06-01 02:22
comments: true
categories: java, android
---

こんにちは。

みけです。

また仰々しいタイトルですみません。

Androidのsdkが5月16日近辺にrevision22に更新されたようです
---

リリースノートを一部抜粋、翻訳(意訳)するとこんな感じです。

+ 既存の`platfomr-tools`を元にSDKの構造を変更して、新しいビルドツール等を追加しました。この変更により、ビルドツールのバージョンとIDEのバージョンを切り離すことが可能になります。その結果、IDEの更新なしにツールの更新が可能になりました。


さて、僕はAndroidに関しては情弱なので、この変更が意味するところをあまり気にせず、

本題である**「JavaがAndroidでない理由」**ことの実例コードを書こうとして

サンプルプロジェクトを作って、実行しようとした時にそれは起こりました。

android-apt-compiler: Cannot run program "/Users/name/Android/sdk/android-sdk-macosx/platform-tools/aapt": error=2, No such file or directory
---

<img src="https://pbs.twimg.com/media/BLlghpZCAAA8qOp.png:large" style="width : 504px;"/>

うん？コンパイル用のツールが見つからない？

というわけで、件のディレクトリーに移動、ファイル探してみると…

```
:~ mike$ cd ~/Android/sdk/android-sdk-macosx/platform-tools/
:platform-tools mike$ ll
total 3664
drwxr-xr-x   7 mike  staff   238B  5 31 19:05 .
drwxr-x---  14 mike  staff   476B  5 31 18:31 ..
-rw-r--r--   1 mike  staff   396K  5 31 18:30 NOTICE.txt
-rwxr-xr-x   1 mike  staff   1.2M  5 31 18:30 adb
drwxr-xr-x   3 mike  staff   102B  5 31 18:30 api
-rwxr-xr-x   1 mike  staff   185K  5 31 18:30 fastboot
-rw-r--r--   1 mike  staff    16K  5 31 18:30 source.properties
```

うむ、確かにそんなものはない…

で、いろいろとディレクトリーを漁ってみると

```
:~ mike$ cd ~/Android/sdk/android-sdk-macosx/build-tools/17.0.0/
:17.0.0 mike$ ll
total 61624
drwxr-xr-x  11 mike  staff   374B  5 31 18:30 .
drwxr-xr-x   3 mike  staff   102B  5 31 18:30 ..
-rw-r--r--   1 mike  staff    11K  5 31 18:30 NOTICE.txt
-rwxr-xr-x   1 mike  staff   1.2M  5 31 18:30 aapt
-rwxr-xr-x   1 mike  staff   273K  5 31 18:30 aidl
-rwxr-xr-x   1 mike  staff   141K  5 31 18:30 dexdump
-rwxr-xr-x   1 mike  staff   2.5K  5 31 18:30 dx
drwxr-xr-x   3 mike  staff   102B  5 31 18:30 lib
-rwxr-xr-x   1 mike  staff    28M  5 31 18:30 llvm-rs-cc
drwxr-xr-x   4 mike  staff   136B  5 31 18:30 renderscript
-rw-r--r--   1 mike  staff    16K  5 31 18:30 source.properties
```

お、あった。

…あったはいいよ。

でも、このディレクトリーは何？

**`17.0.0`**

何、このディレクトリー。

数字ディレクトリー。

**どこのSIerの作業ですか？**

え、じゃあ、君は新しいAPIバージョン(例えば18)が出たら、

**`18.0.0`ディレクトリーでも作ってくれるんです？**

これ、ビルドスクリプトとか、ビルド関連のちょっとした作業とか

ぶっ壊れるよね…え、ぶっ壊れない？なら、いいんだけど、

**IntelliJ IDEA様**を欺いているよね…

これじゃ

Androidって**write once, run once upon a time**ってことだよね
---

Javaの最も貫徹したコンセプトである

write once run anywhere
---

を破っているよね。

というわけで、結論

AndroidはJavaではなかった、決して
---


**p.s.**

最初この問題にあたった時にStack Overflowを調べました。

すでに、先駆者がいました。

[Android Hello-World compile error: Intellij cannot find aapt](http://stackoverflow.com/questions/16588969/android-hello-world-compile-error-intellij-cannot-find-aapt)

で、解決方法のアドバイスとして

+ **JetBrains様がIDEをアップデートしてくれるのを待つしかないんじゃない？**
+ シンボリックリンクを貼ればいいんじゃない
+ ツール類を`platform-tools`にコピペすればいいんじゃない

と有りました。

当面の解決策としては2番目が採択されそうですが、

```
:17.0.0 mike$ ln -s aapt ../../platform-tools/aapt
:17.0.0 mike$ ln -s aidl ../../platform-tools/aidl
:17.0.0 mike$ ln -s lib ../../platform-tools/lib
```

をやった後ですが、


<img src="https://googledrive.com/host/0B4hhdHWLP7RReVgzYzJvOHZJS2s" style="width : 488px;" />

はい、コンパイルできません。

というわけで、JetBrains様がIntelliJ IDEAを対応させるのを待つしかなさそうです。


IntelliJガチ勢の僕としては、googleに一言もの申したい

googleさん、JetBrains様にごめんなさいしてください
---

つーか、ツールがバージョンアップしてもIDEのバージョンアップが必要ないと言ってた、

あのリリースノート何だったんです？

google 「ツールがバージョンアップしてもIDEのバージョンアップがいらないと言ったな、あれは、嘘だ」


{% render_partial _includes/post/post_footer.html %}

