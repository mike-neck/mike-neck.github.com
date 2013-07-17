---
layout: post
title: "Groovyでantログの出力先を変更する"
date: 2013-07-17 14:43
comments: true
image: http://groovy.codehaus.org/images/groovy-logo-medium.png
categories: groovy
---

<img src="http://groovy.codehaus.org/images/groovy-logo-medium.png" style="width : 450px;">

こんにちわ、みけです。

groovyでantのタスクでいろいろと大量のタスクを実行させている時に、

標準出力に出てくるログがちょっとうざったい時があります。

その場合に、ログの出力先のstreamを変更することができるようです。

```groovy
// 出力先のstream
def x = new ByteArrayOutputStream()
// antの出力先を変更 
ant = new AntBuilder()
ant.project.buildListeners.each{ 
    it.outputPrintStream = new PrintStream(x)
} 
// 大量のタスクを実行
(1..100).each {
    ant.echo "this is $it time log."
}
```

このスクリプトを実行すると次のような標準出力になります。

```
1..100
```

実際にはこのstreamを外部のファイルにしておくと後で実行結果だけを参照するなどできます。


{% render_partial _includes/post/post_footer.html %}

