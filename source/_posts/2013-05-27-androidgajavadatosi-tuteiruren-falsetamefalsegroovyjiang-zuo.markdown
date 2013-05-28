---
layout: post
title: "AndroidがJavaだと思っている人のためのGroovy講座 - 1"
date: 2013-05-27 12:29
comments: true
categories: groovy, java
---

タイトルが偉そうなこと書いていますが、

大したことは書きません。

みけです。


Android Studioが出てきてgradleとは何ぞと思っている方がいると思います
===

gradleはgroovyで記述できるビルドシステムです。

antの自由なところと、mavenの依存性管理を組み合わせたイケてるところが売りです。


さて、この記事では…

gradleのことはほとんど書きません
===

Android開発者でgradleとは何やねん？って思っている方には

<iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=B00C7AMKTU" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>

こちらに詳細が書かれていますので、読んでみて下さい。


で、この記事は何を書いているの？というわけで、

簡単なgroovyの文法の説明を書きます
===

これを覚えると、gradleの記述で少し楽をできるかもしれません。

でも、groovyのことを完璧に説明しているかというと、そうでもないので、

詳しくは次の本を買って読んで下さい。

<iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=4774147273" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>


では、以下、本題

メソッドの引数部分の括弧は省略できる
---

多分gradleを初めて見た人で何やねんこれと思う記述はこういうやつだと思います。

```groovy build.gradle
task myTask
```

この`task`というのは、`AbstractProject`という抽象クラスで定義されている`public Task task(Object task)`というメソッドです。このメソッドはタスクをプロジェクトに登録するメソッドです。

したがって、上記の例では`task`メソッドによって`myTask.toString()`で得られる名前(`myTask`)のタスクが作成されます。

gradleの中で同様な記述がたくさんあると思います。

例えば

```groovy build.gradle
repositories {
    // some repository configuration
}
dependencies {
    // some dependency configuration
}
```

これら`repositories`や、`dependencies`も、実はみんなメソッドです。

### 裏をとってみる

では、こっから先はちょっと混みいった話

gradleのprojectの元になるクラスは`org.gradle.api.internal.project.AbstractProject`クラスです。

このクラスの中を少し見てみましょう。

```java AbstractProject.java
public abstract class AbstractProject
        extends AbstractPluginAware
        implements ProjectInternal, DynamicObjectAware {
    // some codes

    public void repositories(Closure configureClosure) {
        ConfigureUtil.configure(configureClosure, getRepositories());
    }

    public void dependencies(Closure configureClosure) {
        ConfigureUtil.configure(configureClosure, getDependencies());
    }

    // some codes

    public Task task(String task) {
        return taskContainer.create(task);
    }

    public Task task(Object task) {
        return taskContainer.create(task.toString());
    }

    // some codes
}
```

はい、`dependencies`や`repositories`、`task`といったキーワードはすべてメソッドですね。


結論：groovyではメソッドの引数部分の括弧を省略できる
---


…ん、gradleのことほとんど書かないと言ったな、あれは嘘だ。