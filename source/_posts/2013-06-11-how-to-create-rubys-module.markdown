---
layout: post
title: "初めてRubyのモジュールを書いてみた"
date: 2013-06-11 02:04
comments: true
categories: ruby
---

こんにちわ、みけです。

前回の記事[『Rubyのmixinの話を読んでいたら、何故かjavaを書いていた』](http://mike-neck.github.io/blog/2013/06/08/rubyfalsemixinfalsehua-wodu-ndeitara%2C-he-gu-kajavawoshu-iteita/)はあまりにも

rubyに対して失礼なので、

初心者らしく、ちゃんと`module`を作ってみました。


ところで、僕の感心事は前回の記事でも書いたように、

rubyの`module`はインスタンス変数にアクセスすることができるのか？
---

ということです。

で、結論から言えば、

rubyの`module`はインスタンス変数にアクセスできました
---

となります。


### `module`を作成

```ruby having_title.rb
# -*- codig: utf-8 -*-

module HavingTitle
  def title=(t)
    @title = t 
  end
  def title
    @title
  end
end
```

`module`の`HavingTitle`は単純にタイトルというものに

アクセスすることができるモジュールです。

先程も書きましたが、僕の疑問は`@title`フィールドに、

インスタンス化することができない`module`がアクセすることができるのか

ということです。


### `module`を`include`した`class`を作成

```ruby book.rb
# -*- coding: utf-8 -*-

require './having_title'

class Book
  include HavingTitle

  attr_accessor :price

  def initialize(title, price)
    @price = price
    self.title = title
  end

  def print_info
    "Book[title : #{@title}, price : #{@price}]"
  end
end
```

タイトルというものを持っていそうなクラスということで

`Book`というクラスをつくりました。

後は`Book`クラスをインスタンス化して、

フィールド`@title`にアクセスできるかどうか確認出来れば

僕の疑問は解決出来ます。


### irbで動作を確認

あ、まだrubyでのテストの書き方知らないので、

ここはirbで勘弁して下さい。

```txt
irb(main):001:0> require './book'
=> true
irb(main):002:0> book = Book.new 'anti-oedipus', 4000
=> #<Book:0x007f90890b6e38 @price=4000, @title="anti-oedipus">
irb(main):003:0> book.print_info
=> "Book[title : anti-oedipus, price : 4000]"
```

というわけで、特に何のエラーも出ることなく、

フィールド`@title`にアクセスできているようですね。


モジュールの機能の一つとしてのMix-in
---

まあ、[『初めてのRuby』](http://www.amazon.co.jp/gp/product/4873113679/ref=as_li_ss_tl?ie=UTF8&camp=247&creative=7399&creativeASIN=4873113679&linkCode=as2&tag=kkkjkrt-22)にはモジュールについて次のように書いてあります。

> **モジュール**はクラスに似ています。モジュールは「インスタンス化できないクラス」のようなものです。
> `Class`クラスは`Module`のサブクラスですから、「クラス = モジュール + インスタンス化能力」と言ってもよさそうです。
> ([Yugui著『初めてのRuby』](http://www.amazon.co.jp/gp/product/4873113679/ref=as_li_ss_tl?ie=UTF8&camp=247&creative=7399&creativeASIN=4873113679&linkCode=as2&tag=kkkjkrt-22)(オライリー・ジャパン、2008年、p.159))

Javaしかわからない(Javaも詳しくはわからない)僕には、

`abstract class`と考えておけばよさそうです。

で、複数のモジュールを継承できるので、

なんか、アレですね…


{% render_partial _includes/post/post_footer.html %}

