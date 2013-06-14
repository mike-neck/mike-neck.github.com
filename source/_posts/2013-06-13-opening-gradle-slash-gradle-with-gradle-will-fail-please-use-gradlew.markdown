---
layout: post
title: "gradleプロジェクトは、gradleを使うよりも、gradlewを使うことがオススメです"
date: 2013-06-13 02:41
comments: true
categories: [groovy, java]
---

<img src="http://www.gradle.org/img/gradle_logo.gif"/>

みけです。

最近、Java開発者のgradleへの注目には眼を見張るものがあります。

gradle1.0-m2くらいから使っていた僕も、

参考になるような記事がたくさんあります。

みなさんありがとうございます。


gradleのすばらしさ
===

僕はgradleいいよという時に、いつも聞かれるのですが、

### AntとMavenにくらべてgradleの何がいいの？

という質問があります。


DSLで書けるとか、

dependencyの指定が簡単とか、

まあ、各種ツールに精通すれば、

別にantでもmavenでも変わりません。


あえてそれでもgradleを選ぶ理由を上げるなら、

**gradle wrapper**の存在です。


チームで開発する場合、開発環境をチーム内で同じにしておくことが重要です。

メンバーの開発マシンの

antのバーション、mavenのバージョン、

これらが違ったということでビルドに失敗して、

その原因追求のために時間がかかるということも

よくある話です。


一方で、gradleでは

**gradle wrapper**を用いれば、

ビルドツールのバージョンに振り回されることはありません。


あまりピンとこないかもしれませんが、

実例をあげてみます。


gradle本体をIntelliJ IDEAに取り込む
---

gradle DSLを読んでいて、どうしてもわからないことがある場合、

gradle本体のソースを読むことがあります。

最新(2013/06/12時点)のソースを落としてきて、

`gradle idea`タスクを実行してみます。

```
$ gradle --version

------------------------------------------------------------
Gradle 1.6
------------------------------------------------------------

Gradle build time: 2013年5月7日 9時12分14秒 UTC
Groovy: 1.8.6
Ant: Apache Ant(TM) version 1.8.4 compiled on May 22 2012
Ivy: 2.2.0
JVM: 1.7.0_13 (Oracle Corporation 23.7-b01)
OS: Mac OS X 10.8.4 x86_64

$ gradle idea
:buildSrc:clean
:buildSrc:compileJava UP-TO-DATE
:buildSrc:compileGroovy
:buildSrc:processResources
:buildSrc:classes
:buildSrc:jar
:buildSrc:assemble
:buildSrc:checkstyleMain
:buildSrc:compileTestJava UP-TO-DATE
:buildSrc:compileTestGroovy
:buildSrc:processTestResources
:buildSrc:testClasses
:buildSrc:checkstyleTest UP-TO-DATE
:buildSrc:codenarcMain
:buildSrc:codenarcTest
:buildSrc:test
:buildSrc:check
:buildSrc:build

FAILURE: Build failed with an exception.

* Where:
Script '/Users/mike/IdeaProjects/gradle/gradle/gradle/integTest.gradle' line: 32

* What went wrong:
A problem occurred evaluating script.
> Failed to notify action.
   > Could not find property 'reports' on task ':announce:integTest'.

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 54.695 secs
```

最新のgradle(バージョン1.6)でエラーが発生します。

ビルドスクリプト:buildSrcプロジェクトのテスト中に落ちてしまいます。

一方、プロジェクトに付属している**wapper**で実行してみます。

```
$ ./gradlew --version

------------------------------------------------------------
Gradle 1.7-20130519231153+0000
------------------------------------------------------------

Build time:   2013-05-19 23:11:53 UTC
Build number: none
Revision:     9a7199efaf72c620b33f9767874f0ebced135d83

Groovy:       1.8.6
Ant:          Apache Ant(TM) version 1.8.4 compiled on May 22 2012
Ivy:          2.2.0
JVM:          1.7.0_13 (Oracle Corporation 23.7-b01)
OS:           Mac OS X 10.8.4 x86_64

$ ./gradlew idea
Deleting directory /Users/mike/.gradle/wrapper/dists/gradle-1.7-20130519231153+0000-bin/1cbtqldhq0muu2cto5pdcq66ee/gradle-1.7-20130519231153+0000
Unzipping /Users/mike/.gradle/wrapper/dists/gradle-1.7-20130519231153+0000-bin/1cbtqldhq0muu2cto5pdcq66ee/gradle-1.7-20130519231153+0000-bin.zip to /Users/mike/.gradle/wrapper/dists/gradle-1.7-20130519231153+0000-bin/1cbtqldhq0muu2cto5pdcq66ee
Set executable permissions for: /Users/mike/.gradle/wrapper/dists/gradle-1.7-20130519231153+0000-bin/1cbtqldhq0muu2cto5pdcq66ee/gradle-1.7-20130519231153+0000/bin/gradle
:buildSrc:clean
:buildSrc:compileJava UP-TO-DATE
:buildSrc:compileGroovy
:buildSrc:processResources
:buildSrc:classes
:buildSrc:jar
:buildSrc:assemble
:buildSrc:checkstyleMain
:buildSrc:compileTestJava UP-TO-DATE
:buildSrc:compileTestGroovy
:buildSrc:processTestResources
:buildSrc:testClasses
:buildSrc:checkstyleTest UP-TO-DATE
:buildSrc:codenarcMain
:buildSrc:codenarcTest
:buildSrc:test
:buildSrc:check
:buildSrc:build
:ideaModule

## 途中長いので略

:ui:ideaModule
:ui:idea
:wrapper:buildReceiptResource
:wrapper:ideaModule
:wrapper:idea

BUILD SUCCESSFUL

Total time: 2 mins 11.981 secs
```

という形で、

マシンにインストールされているgradleのバージョンの如何にかかわらず、

プロジェクトごとに最適化されたビルドを提供してくれます。


{% render_partial _includes/post/post_footer.html %}

