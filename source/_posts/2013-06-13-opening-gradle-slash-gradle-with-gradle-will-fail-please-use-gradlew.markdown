---
layout: post
title: "gradleプロジェクトを開くときは、gradleを使うよりも、gradlewを使うことがオススメです"
date: 2013-06-13 02:41
comments: true
categories: [groovy, java]
---

みけです。

最近、Java開発者のgradleへの注目には眼を見張るものがあります。

gradle1.0-m2くらいから使っていた僕も、

参考になるような記事がたくさんあります。


一方、僕の方は、最近gradleに関するポストが少ないです。


groovyの理解なくしては読みやすいgradleなんて書けない
===

僕は元々原理主義的な考え方の持ち主で、

基本を押さえていないと、すごい不安を感じるタイプの人間なので、

(「なんかわからないけど動いた」とかいうのがすごい苦手)

簡単なgradleのビルドスクリプトを書くのにも、

gradleの公式ドキュメント類

+ [gradle DSLs](http://www.gradle.org/docs/current/dsl/)
+ [gradle Users Guide](http://www.gradle.org/docs/current/userguide/userguide.html)
+ [gradleそのもののソース](https://github.com/gradle/gradle)

さらには

+ [groovyそのもののソース](https://github.com/groovy/groovy-core)

を読むことが多いです。

あ、でもこれってふつうのコトですよね。

gradleはDSLですので、

読みやすいビルドスクリプトであるということを目指しています。

そして、このDSLをサポートするための言語として、

groovyが選ばれたわけです。

だから、誰もが読みやすい、簡潔な記述を目指すために

groovyをしっかりと抑えること、

これは欠かせないことだと思います。


まあ、ビルドスクリプトをガチガチのJavaで書いても

問題はありませんけどね…

これは、あくまで個人的な思想・信条レベルでの話です。


gradleというかソースを読むのにIDEは欠かせない
===

さて、そういうわけで、gradleのソースを読んでいくわけですが、

ソースを読むのにIDEは欠かせません。

+ フィールドとして使われているクラスを参照する
+ 呼び出されているメソッドを参照する
+ インターフェースの実装クラスに移動する(これ重要)


{% render_partial _includes/post/post_footer.html %}

