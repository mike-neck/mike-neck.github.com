---
layout: post
title: "Java8 lambda式を使ってみた"
date: 2013-06-27 22:24
comments: true
categories: java
---

こんにちわ、みけです。

Java8初心者勉強会というのを開催してみました。

参加者2人でした。

Java8はだれも興味ないんだなと思いました。


Java8のラムダ式
---

特にもうこれといって目新しいこともありません。

メソッドが一つだけのインターフェースを記述するときに、

非常に記述が楽になるというものです。

例えば次のようなクラスがあるとします。

```java Item.jara
public class Item {
    final String name;
    final int price;
    public Item(String name, int price) {
        this.name = name;
        this.price = price;
    }
    public String getName() {
        return name;
    }
    public int getPrice() {
        return price;
    }
}
```

上記の`Item`クラスを`price`の昇順、`name`の昇順でソートするコードは以下のようになります。

```java
List<Item> items = getItemList();
items.sort((left, right) -> {
    int priceOrder = left.getPrice() - right.getPrice();
    int nameOrder = left.getName().
            compareTo(right.getNmae());
    return priceOrder != 0? priceOrder : nameOrder;
});
```

ところで、`Item`クラスの`price`だとか`name`だとかについて、

それをソートするという操作は別に外のクラスが実装しても構わないけど、

`Item`クラスが持っている方が何かと便利です。

したがって、オーダーするにあたって、`Item`クラスに次のようなメソッドを

持たせるようにします。

```java Item.java
public class Item {
    final String name;
    final int price;
    // 途中省略
    public int comparePriceAscNameAsc (Item that) {
        int priceOrder = this.price - that.price;
        return priceOrder != 0 priceOrder :
                this.name.compareTo(that.name);
    }
}
```

先ほどのソートをするコードのラムダ式部分は非常に簡単化されます。

```java
items.sort((left, right) -> left.comparePriceAscNameAsc(right));
```

ところで、呼び出されるメソッドはこのケースの場合わかりきっているので、

> TODO : この記述は適当に書いているのでドキュメントを読み直します

Method Referenceに変更することが可能です。


```java
items.sort(Item::comparePriceAscNameAsc)
```

という感じで、ラムダ式っぽい記述はなくなりました。

ちなみにMethod and Constructor Referenceは[Lambdaの仕様の一部](http://cr.openjdk.java.net/~dlsmith/jsr335-0.6.1/C.html)です。


{% render_partial _includes/post/post_footer.html %}

