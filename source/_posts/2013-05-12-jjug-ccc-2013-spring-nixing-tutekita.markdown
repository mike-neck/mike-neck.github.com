---
layout: post
title: "JJUG CCC 2013 Spring に行ってきた"
date: 2013-05-12 16:47
comments: true
categories: [java, hackathon]
---

みけです。

2013/05/11行われた[JJUG CCC 2013 Spring](http://www.java-users.jp/?page_id=330)に午後から行って来ました。

なんか午前の基本講演のJim Weaverさんと

[JavaFX Advent Calendar2012](http://atnd.org/events/33874)をやった人達で

ランチに行ってきたそうです。

午前に行けなくて残念…

以下、僕が参加したセッションのまとめ

H-1 Java EE 6 から Java EE 7 に向かって
---

王子こと寺田さんのセッション。

まあ、大体聞いたことのある話でした(失礼！)。

+ JavaEE7に向けてJavaEE6も勉強して下さい。
+ JavaEE6はJavaEE5以前に比べてかなり軽量化されています。
+ JavaEE6からはテストも非常に容易になっています。

といった感じです。

その他、JavaEE7に追加される機能の一部の概要の説明がありました。

### 結論

+ JavaEE7はおもろいので、ぜひやろうぜ
+ JavaEE6やってない人はJavaEE6からどうぞ
+ 資料公開してくれるはず、はず、はず

<iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=4798124605" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>



H-2 H-2 Project Lambda Essential
-----

桜庭さんのProject Lambdaに関するセッション。

まあ、ラムダは大体わかっているし、あれでしたが、

Streamに関する話は面白かったです。

### Streamとは？

終端の定義されていないCollectionのことだそうです。

次のようなクラスが定義されています。

+ Object用 `java.util.stream.Stream<T>`
+ int用 `java.util.stream.IntStream`
+ long用 `java.util.stream.LongStream`
+ double用 `java.util.stream.DoubleStream`

#### メソッドとか

メソッドとかの例を上げてみます。

ちなみに、昔(今)どうやっているかも一緒に書きます。


##### filter

```java filter
IntStream stream = IntStream.range(1,10);
int[] result = stream.filter(x -> x % 2 == 0).toArray();
// result -> [2, 4, 6, 8, 10]

```


昔のやり方

```java filter-old-style
List<Integer> list = new ArrayList<>();
List<Integer> result = new ArrayList<>();
for (int i = 1; i <= 10; i ++) {
    list.add(i);
}
for (int item : list) {
	if (item % 2 == 0) {
		result.add(item);
	}
}

```

##### map

```java map
IntStream stream = IntStream.range(1,4);
int[] result = stream.map(x -> x * x).toArray();
// result -> [1, 4, 9, 16]

```

ちなみに昔のやり方

```java map-old-style
List<Integer> list = new ArrayList<>();
List<Integer> result = new ArrayList<>();
for (int i = 1; i <= 4; i ++) {
    list.add(i);
}
for (int item : list) {
    list.add(item * item);
}

```

##### reduce

```java reduce
IntStream stream = IntStream.range(1,10);
int result = stream.reduce(0, (x, y) -> x + y).get();
// result -> 55

```

ちなみに昔のやり方

```java reduce-old-style
List<Integer> list = new ArrayList<>();
int result = 0;
for (int i = 1; i <= 10; i ++) {
    list.add(i);
}
for (int item : list) {
    result += item;
}

```

#### 対応

+ filter -> if文
+ map -> getter
+ reduce -> 副作用プログラミング

### Project Lambdaに関して

#### lambdaを使用しているのにHoge$A.classみたいなものができない

lambda式が表しているクラスは動的に評価されてinvoke dynamicで実行される

#### mapとreduceを実行しているのに、ループが一回しかまわらない

map/reduce式は遅延して実行される

### まとめ

+ Project LambdaとかStreamは記法がかなり変わるので、使用禁止と言われないように勉強しろ(とくにプロジェクトの<strike>エロイ</strike>エライ人)

<iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=0321927761" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>



R5-3 Type Annotation って何? それを使うとプログラムはどう変わる?
----

Type Annotationに関するセッション

### 比較

+ これまでのアノテーション(AnnotationType)は宣言をするアノテーション
+ TypeAnnotationは型、参照の利用に関する説明のアノテーション

### 例

```java TypeAnnotation-example
// 空リストではない、読み取り専用のリスト
@NotEmpty @ReadOnly List<String> customer = CustomerService.getCustomer().withMultipleAccount();

// nullなkey不可、空リストでなく読み取り専用なvalueを持つマップ
Map<@NonNull String, @NotEmpty List<@ReadOnly Text>> document = DocumentService.splitDocumentToTextList(doc);
```

ここでなんとかServiceとか書いていますが、テキトーなので突っ込まないで下さい。


### 記述位置で意味が異なる

```java position-has-much-meanings
// 読み取り専用のリスト
// customers.add(customer)や、customers.replace(index, customer)はできない。
@ReadOnly List<Customer> customers = CustomerService.getCustomer().withMultipleAccount();

// 変更可能なリスト
// ただし入っているCustomerへの変更は不可
// customer.addNewAddress(address)などは実行できない
List<@ReadOnly Customer> customers = CustomerService.getCustomer().withMultipleAccount();
```

### 使えない場所

+ クラスリテラル
+ import文
+ staticメンバー

### 簡単な例

+ `@NonNull`
+ `@NotEmpty`
+ `@Interned`
+ `@ReadOnly`
+ `@Critical`


### 残念なこと

+ 書けるけど、意味は無い…
+ JavaSE8で事前定義された型アノテーションは(今のところ)ない
+ Java SE8でも入らないし、Java SE9でも入らないかもしれない
+ 単純に実装者が気をつけろよレベル

### まとめ

うむ、なんかちょっと残念。

<iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=0321927761" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>



H-4 失敗から学ぶAPI設計
----

イケメンことゆーすけさんのセッション

Twitter4Jのプロモーションとか、たくさん使ってもらうためにしたことなどを発表

### たくさん使ってもらうためのプロモーション的な

+ 汚いことでもなんでもする
+ 手厚いサポートをする。ggrksとかしない

### たくさん使ってもらうための技術的な

+ インターフェースを使わないでクラスを使う
+ あまりパッケージを増やさない
+ 名前はよく考える
+ デザパタとかも使わせない
+ 継承させない
+ 本当に使ってほしくないクラスは異様な名前にする

あとは、資料が公表されているので、そちらを参照。

<iframe src="http://www.slideshare.net/slideshow/embed_code/20968265" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC;border-width:1px 1px 0;margin-bottom:5px" allowfullscreen webkitallowfullscreen mozallowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="http://www.slideshare.net/yusukey/api-ccch4-jjug-jjugccc-jjug-ccc-2013-spring" title="失敗から学ぶAPI設計 #ccc_h4 #jjug #jjug_ccc JJUG CCC 2013 Spring " target="_blank">失敗から学ぶAPI設計 #ccc_h4 #jjug #jjug_ccc JJUG CCC 2013 Spring </a> </strong> from <strong><a href="http://www.slideshare.net/yusukey" target="_blank">Yusuke Yamamoto</a></strong> </div>

<iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=477414732X" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>



R5-5 [BOF] Java読書会ライブ
---
 
実は昔デブサミのLT大会でLTを聞いてから興味を持っていたので、聞いてみたかったセッション
 
実際に音読をしているし、途中で議論をやっていて面白そうでした。

今は[JUnit実践入門](http://www.amazon.co.jp/gp/product/477415377X/ref=as_li_ss_tl?ie=UTF8&camp=247&creative=7399&creativeASIN=477415377X&linkCode=as2&tag=kkkjkrt-22)を読んでいるそうです。

### 概要

+ 月一回土曜日に開催
+ まず自己紹介と近況の報告をする
+ 書記を選出する
+ 書記は議論のポイントなどをまとめてサイトに掲載する
+ 午前10:00くらいから昼間でと午後5時まで続ける
+ その後は有志で飲みに行く

まあ、普通の勉強会ですね。

 
### 質疑応答

#### 読む本はどうやって決定しているか？

+ 投票によって決定しているそうです。
+ なお、投票は普通の参加者は1票、書記は2票を入れることができるそうです。

#### どれくらいのスピードで読んでいるのか

+ 毎回約60ページ程度読んでいます。

#### 予習とか必要か？

+ してないけど、してくる人もいる

#### コードの実行をしたりするのか？

+ パソコン持ってきていてやってくれる人もいる

#### どんな本が読書会に向いているか

+ 入門本よりはすこし難しめな本で、どうしても積ん読になってしまうようなもののほうが適している
+ いろんなコンテキストを持っている人が参加するので、それが勉強になる


### まとめ

面白そうなので、一回参加してみたいと思いました。

参考リンク : [Java読書会BOFのページ](http://www.javareading.com/bof/)



R2-6 [BOF] 地方における勉強会事情
----

沖縄、大都会岡山、札幌＋急遽大阪のコミュニティ・勉強会の主催者によるパネルディスカッション

東京の勉強会との違いがわかる一方で、

Javaの勉強会が抱えている問題と共通するものがあるなーという感じがしました。

詳しくはtogetterにまとめられていますので、

そちらを参照して下さい。

[JJUG CCC 2013 Spring R2-6 [BOF] 地方における勉強会事情](http://togetter.com/li/501590)

> 2013/05/12 19:54 追記
> 
> 地方の勉強会で初参加する人は
> 
> 勉強会に参加してきたというブログを読んで
> 
> いけそうだなと思って参加するらしいです。
> 
> したがって、勉強会の活性化をしたいなら
> 
> + **とにかくブログを書け**
> + **勉強会はブログを書くまでが勉強会です**
> 

JJUG CCC 2013 Spring 懇親会
---

一応参加して来ました。

そしてなぜかLTさせられることになったので、

グダグダなやつをやってきました。

参考 : [ゆとりさんが鮨を奢ってくれるそうなので、感謝の気持を込めて、たくさんのプロセスに「sushi」と言わせてみた](http://mikeneck.blogspot.jp/2013/05/sushi.html)



まとめ
---

Java楽しいですね。

僕はもう人間的にかなり不活性ですが、

Java自体は活性化して貰いたいので、

なんか継続できる勉強会でもやりたいなと思います。

{% render_partial _includes/post/post_footer.html %}

