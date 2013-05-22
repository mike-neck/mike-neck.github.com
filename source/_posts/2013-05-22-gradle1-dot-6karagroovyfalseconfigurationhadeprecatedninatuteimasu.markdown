---
layout: post
title: "gradle1.6からgroovyのconfigurationはdeprecatedになっています"
date: 2013-05-22 20:08
comments: true
categories: groovy
---

gradle1.6からgroovyのconfigurationは非推奨になっています
---

まあ今さらですが、さきほどbuild.gradleを書いた時に、

```
$ gradle idea
The groovy configuration has been deprecated and is scheduled to be removed in Gradle 2.0. Typically, usages of 'groovy' can simply be replaced with 'compile'. In some cases, it may be necessary to additionally configure the 'groovyClasspath' property of GroovyCompile and Groovydoc tasks.
:ideaModule
Download http://repo1.maven.org/maven2/org/codehaus/groovy/groovy-all/2.1.3/groovy-all-2.1.3.pom
Download http://repo1.maven.org/maven2/org/codehaus/groovy/groovy-all/2.1.3/groovy-all-2.1.3-sources.jar
Download http://repo1.maven.org/maven2/org/codehaus/groovy/groovy-all/2.1.3/groovy-all-2.1.3.jar
:ideaProject
:ideaWorkspace
:idea

BUILD SUCCESSFUL

Total time: 11.174 secs
$ 
```

と表示されたので、「うぉっ」と思ってドキュメントを読んでみました。

[groovy configuration is deprecated](http://www.gradle.org/docs/current/release-notes#groovy-configuration-is-deprecated)


じゃあ、今後どうするのかというと、

compile configurationにgroovyのartifactを指定する
===

ということです。

つまり、これまでは

```groovy build.gradle
dependencies {
    groovy : 'org.codehaus:groovy:groovy-all:2.1.3'
}
```

と書いていましたが、

```groovy build.gradle
dependencies {
    compile : 'org.codehaus.groovy:groovy-all:2.1.3'
}
```

と書けばよいようです。

<table>
<tbody>
<tr>
<td><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=B009X5KIFK" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
<td><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=3864900492" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
<td><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=1617291307" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
</tr>
</tbody>
</table>

