---
layout: post
title: "gradleのjavadocタスクでjavadocを英語で出力する方法"
date: 2013-06-13 22:39
comments: true
categories: [java, gradle, groovy]
---

<img src="http://www.gradle.org/img/gradle_logo.gif"/>

こんにちわ、みけです。

表題の件はこの記事の前半部分に書いています。

なので、必要な人は前半部分だけ読んで下さい。

---

大した話ではありませんが、

gradleのjavadocタスクで出力されるjavadocが日本語で出力される
===

のがちょっと残念な時があります。


`java`プラグインを入れていれば、`javadoc`タスクが自動で追加されます。

僕のような日本語環境でやっている人だと、

頑張ってjavadocを英語で書いても、

テンプレートが日本語で出力されてしまいます。


会社で日本語を使っていて、

javadocが日本語でないと困る場合は、

全然問題ないとおもいますが…


オープンソースなソフトウェアを開発している場合、

javadocが日本語だとなんか若干困ります。

(まあ、だいたいjavadocのテンプレートなんで、何が出力されているかなんてわかりますけどね…)


`javadoc`タスクの設定で`options.locale`に`en_US`を指定すればいいです
===

つまり、以下のとおりになります。

```groovy build.gradle
apply plugin : 'java'

javadoc {
    options.locale = 'en_US'
}
```

**本題、終わり**


---

ちなみに、僕の得意技はtypoなので、間違えてこんなビルドスクリプト書いてました。

**(誤)**

```groovy build.gradle
apply plugin : 'java'

javadoc {
    options.local = 'en_US'
}
```

これを実行したら、こんなエラーが出力されました。

```
$ gradle javadoc
Picked up _JAVA_OPTIONS: -Dfile.encoding=UTF-8

FAILURE: Build failed with an exception.

* Where:
Script '/Users/mike/myprojects/sample/build.gradle' line: 4

* What went wrong:
A problem occurred evaluating script.
> No such property: local for class: org.gradle.external.javadoc.StandardJavadocDocletOptions
  Possible solutions: locale

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 8.941 secs
```

gradle曰く

> `local`なんてオプションないよ、`locale`じゃない？

というわけで、

gradleさん正しい答えを教えてくれたりと、親切ですね。
===

本当に親切なツールです。

しかし、

本当に親切なのはgroovyの思想
===

です。

例えば次のようなgroovyのスクリプトがあるとします。

```groovy error.groovy
def range = 1..10
range.colect {
    println (it * it)
}
```

これを実行するとこういうエラーが出力されます。

```
$ groovy error.groovy
Caught: groovy.lang.MissingMethodException: No signature of method: groovy.lang.IntRange.colect() is applicable for argument types: (error$_run_closure1) values: [error$_run_closure1@639d564]
Possible solutions: collect(), collect(), collect(groovy.lang.Closure), collect(groovy.lang.Closure), collect(java.util.Collection, groovy.lang.Closure), collect(java.util.Collection, groovy.lang.Closure)
groovy.lang.MissingMethodException: No signature of method: groovy.lang.IntRange.colect() is applicable for argument types: (error$_run_closure1) values: [error$_run_closure1@639d564]
Possible solutions: collect(), collect(), collect(groovy.lang.Closure), collect(groovy.lang.Closure), collect(java.util.Collection, groovy.lang.Closure), collect(java.util.Collection, groovy.lang.Closure)
	at error.run(error.groovy:2)
```

存在しないメソッドを呼び出した時に、

**Possible solutions**ということで、

サジェストしてくれます。

また、有名な例ですが、Power Assertもあります。

```groovy error.groovy
def list = [1,1,2,3,4,5]
assert list - [1,3,6] == [1,2,4,5]
```

これを実行すると、このように表示されます。

```
$ groovy error.groovy
Caught: Assertion failed: 

assert list - [1,3,6] == [1,2,4,5]
       |    |         |
       |    [2, 4, 5] false
       [1, 1, 2, 3, 4, 5]

Assertion failed: 

assert list - [1,3,6] == [1,2,4,5]
       |    |         |
       |    [2, 4, 5] false
       [1, 1, 2, 3, 4, 5]

	at error.run(error.groovy:2)
```

ただ、求めている結果と実際の結果が**違う**という表示だけでなく、

実際の値を示してくれます。

つまり一言で言えば、

groovyの半分は優しさでできています
===

なお、この機能はもともとからgroovyにあったわけではなく、

Spockというテスティングフレームワークから採用された機能です。

groovyの思想では、こういった、(・∀・)ｲｲﾈ!!な機能を

どんどん取り込んでいくというのがあると思っています。


まあ、元々、javaを良い感じで書きたいといった思想から生まれている言語ですし、

実際に、rubyなど他の言語のいいところを借りたりしているので、

後発の優位性を遺憾なく発揮しているわけですが…


{% render_partial _includes/post/post_footer.html %}

