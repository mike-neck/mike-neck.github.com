---
layout: post
title: "rubyのmixinの話を読んでいたら、何故かjavaを書いていた"
date: 2013-06-08 21:22
comments: true
categories: java
---

こんにちわ、みけです。

[@suginoy](https://twitter.com/suginoy)さんに

[『楽しい開発スタートアップRuby』](http://www.amazon.co.jp/gp/product/4774151661/ref=as_li_ss_tl?ie=UTF8&camp=247&creative=7399&creativeASIN=4774151661&linkCode=as2&tag=kkkjkrt-22)を買っていただいたので、

今日はそれを某社オフィスで読んでいました。

Rubyのmix-inについての読んだ後の会話
===

+ 僕「mix-inよくわからん」
+ イケメン「多重継承できるやつでしょう」
+ 僕「mix-inってインスタンス変数とかにアクセスできるのかな？」
+ イケメン「できそうじゃね」
+ 僕「あ〜、Java8のdefault実装でもmix-inと同じ事できそうじゃね」
+ イケメン「いや、できないでしょう。default実装といっても、interfaceにフィールドを持つことができないわけだから」
+ 僕「え、できないのか、あー、そうかできないな。えっ、じゃあ、あれ、誰得なの？」

(多少脚色有り、あと会話内容忘れたので、結構適当)

で、

書いていたのがJava8のコードでした
===

```java Score.java
package org.mikeneck.jdk8;

/**
 * @author mike
 */
public class Score {

    final private String title;

    final private int value;

    public Score(String title, int value) {
        this.title = title;
        this.value = value;
    }

    public String getTitle() {
        return title;
    }

    public int getValue() {
        return value;
    }
}
```

```java Grade.java
package org.mikeneck.jdk8;

import java.util.List;

/**
 * @author mike
 */
public interface Grade {

    public List<Score> scores();

    default long getTotalScore () {
        return scores()
                .stream()
                .mapToLong(Score::getValue)
                .reduce(0l, Long::sum);
    }
}
```

ところで、このdefaultメソッド、

書いている途中はこんな感じでした。


<img src="https://googledrive.com/host/0B4hhdHWLP7RRYk5mVkd1RFFQYmc" style="width : 501px;" />

IntelliJ IDEAではシンタックスをもっとよくできる場合は、

こういう形で通知してくれます。

紫の部分にカーソルをあてて、

Alt + Enter (mac の場合は option + enter)をすると、

操作の候補が表示されます。

<img src="https://googledrive.com/host/0B4hhdHWLP7RRQzhXZnc0Z2lVc0k" style="width : 501px;"/>

ここでは**Replace lambda with method reference**を選択します。

<img src="https://googledrive.com/host/0B4hhdHWLP7RRbnFGWmE1czJrRk0" style="width : 501px;"/>

すごいシンプルなコードになりました。

続いて、mapした結果をreduceしていきます。

<img src="https://googledrive.com/host/0B4hhdHWLP7RRa0s1ZldObzA0VzA" style="width : 501px;"/>

初期値`0l`で、結果を合計したいので、次のような演算を書きます。

<img src="https://googledrive.com/host/0B4hhdHWLP7RRTVAxQUE0d0tQMU0" style="width : 501px;"/>

これはこれでまちがいでありません。

ところで、`java.util.stream.Stream#mapToLong`の戻り値である、

`java.util.stream.LongStream`のjavadocを読むとこのように記述されています。

> Api note : Sum, min, max, and average are all special cases of reduction.
> Summing a stream of numbers can be expressed as:
> 
> ```
> long sum = integers.reduce(0, (a, b) -> a+b);
> ```
> 
> or more compactly:
> 
> ```
> long sum = integers.reduce(0, Long::sum);
> ```
> 
> _引用元 : javadoc (build-89)_

つまり、`java.util.function.LongBinaryOperator`の記述

```
(sum, item) -> sum + item
```

は、`java.lang.Long`の`static`なメソッド`sum(long, long)`に

置き換えることができるということです。

その結果、先のコードは次のようになりました(既出)。

<img src="https://googledrive.com/host/0B4hhdHWLP7RRS25qWnVZX2dHdW8" style="width : 501px;"/>

さて、ここらへんの`Stream`系の操作がエンタープライズな現場で使われるかどうかは

微妙ですが…(というのも、値の集計をするというのであれば、DBにさせたほうが早いので)

実際に現場で使うとなると、

高機能なIDE(`Long::sum`の部分はIntelliJでも簡略化できなかった)と

`Stream`系のApiの書き方を覚えておかないと、

相当効率悪くなるとおもいます。

というわけで、Java8の勉強をしたい方は是非

6月26日(水)の『Java8初心者勉強会』にご参加下さい。

Java8初心者勉強会 - [http://atnd.org/event/java8beginner20130626tokyo](http://atnd.org/event/java8beginner20130626tokyo)

結論

あれ、rubyのmix-inについての話はどこ行った…
===

vimでrubyのコード書くの辛いです…

{% render_partial _includes/post/post_footer.html %}

