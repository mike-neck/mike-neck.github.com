---
layout: post
title: "JavaFXをheadless browserとして使うためのJSObjectの扱い方 - 1"
date: 2013-07-01 13:03
comments: true
categories: java
---

こんにちわ。

みけです。

JavaFXでjavascriptのテスティングフレームワークを作ろうと思ってから、

早1年半。

全然成果があがっていません。

今日は、そんな自分のための俺得なエントリー。

<img src="//googledrive.com/host/0B4hhdHWLP7RRbkltQVpjMUkzNE0" style="width : 450px;"/>

Goal
---

#### 期待していいこと

+ JSObjectでのarrayの取り扱い


#### 期待できないこと

+ JavaFXのスレッドの同期方法
+ JSObjectでのobjectの取り扱い


JSObjectの扱い方 - array編
---

`JSObject#getSlot(int)`を使ってarrayの要素を取得します。

```java
final String script = "(function(){return ['a', 2, 'c', 4, 'e'];})()";
final BlockingQueue<JSObject> queue = new LinkedBlockingQueue<>();
Platform.runLater(() -> {
    queue.put((JSObject) webEngine.executeScript(script));
});
JSObject array = queue.take();
for (int i = 0; i < 5; i++) {
    System.out.println(i + " -> " + array.getSlot(i));
}
```

結果は次のようになります。

```
0 -> a
1 -> 2
2 -> c
3 -> 4
4 -> e
```

ちなみに要素のindexより多い数を`JSObject#getSLot(int)`の引数に渡すと、

`String`の`undefined`が返ってきます。


{% render_partial _includes/post/post_footer.html %}

