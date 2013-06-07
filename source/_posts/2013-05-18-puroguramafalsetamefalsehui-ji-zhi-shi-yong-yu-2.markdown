---
layout: post
title: "プログラマーのための会計知識・用語 - (2)"
date: 2013-05-18 15:04
comments: true
categories: [account, groovy]
---
普通のエンタープライズ系プログラマーであれば、会計用語はちゃんと英語で覚えているものですよね(白目
---

{% img http://img5.blogs.yahoo.co.jp/ybi/1/9f/9a/fpdxw092/folder/1455483/img_1455483_52010810_1?1205033611 %}


僕は[前](http://mike-neck.github.io/blog/2013/05/13/puroguramafalsetamefalsehui-ji-zhi-shi-yong-yu-1/)にも言ったとおりに、

出来損ないのエンタープライズ系プログラマーなので、

アメリカの中学生向けの会計の簡単な本で勉強しています。

<iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=0071779752" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>

この本の第二章に書かれていることをまとめてみます。

会計の基本用語
---

+ 勘定 : **Account**
+ 勘定科目 : **Account Titile**
+ 借方 : **Debit**
+ 貸方 : **Credit**
+ 総勘定元帳 : **Ledger**
+ 勘定科目一覧 : **The Chart of Accounts**
+ 仕訳 : **Journal**
+ 複式簿記 : **Double-Entry Bookkeeping**
+ 試算表 : **Trial balance**

#### [前回の復習](http://mike-neck.github.io/blog/2013/05/13/puroguramafalsetamefalsehui-ji-zhi-shi-yong-yu-1/)

+ 資産 : **Assets**
+ 負債 : **Liabilities**
+ 資本 : **Capital**


借方と貸方の決まり事 (**Convention**)
---

エンタープライズ系プログラマーなら当然わかっていると思いますが…

勘定科目(**Account Title**)は借方科目なのか貸方科目なのか決まっていますね。

#### 借方系の勘定 (**The Account to be debited**)

+ 資産(**Asset**)の増加
+ 費用(**Expense**)の増加
+ 負債(**Liabilities**)の減少
+ 資本(**Capital**)の減少
+ 収入(**Income**)の減少

#### 借方系の勘定 (**The Account to be credited**)

+ 負債(**Liabilities**)の増加
+ 資本(**Capital**)の増加
+ 収入(**Income**)の増加
+ 資産(**Asset**)の減少
+ 費用(**Expense**)の減少



会計のドメインモデルを作ってみる
---

ここまで来たら、もう会計のドメインモデル作れますね。

#### 勘定

```groovy Account.groovy
@Cannonical
class Account {
    String title
    AccountChart accountChart
}
```

#### 取引
```groovy Transaction.groovy
@Cannonical
class Transaction {
    long id
    Date date
    Debit debit
    Credit credit
    String memo
}
```

#### 借方

```groovy Debit.groovy
@ToString
class Debit {
    Account item
    int value
    String memo
}
```

#### 貸方

```groovy Credit.groovy
@ToString
class Credit {
    Account item
    int value
    String memo
}
```

借方と貸方でクラスの構造が全く同じなので、

訓練されたSIer諸氏に於いては同じクラスを使おうと言い出すとおもいます。

しかし、java-ja.dddで増田さんがおっしゃっていたとおり、

借方と貸方は別物なので、別のクラスにします。

[java-ja.dddのまとめについてはこちら](https://gist.github.com/yamashiro/5230921)

#### 勘定科目

```groovy AccountChart.groovy
enum AccountChart {
    ASSET{
        AccountConvention increase() {AccountConvention.DEBIT}
        AccountConvention decrease() {AccountConvention.CREDIT}
    }, EXPENSE{
        AccountConvention increase() {AccountConvention.DEBIT}
        AccountConvention decrease() {AccountConvention.CREDIT}
    }, LIABILITIES{
        AccountConvention increase() {AccountConvention.CREDIT}
        AccountConvention decrease() {AccountConvention.DEBIT}
    }, CAPITAL{
        AccountConvention increase() {AccountConvention.CREDIT}
        AccountConvention decrease() {AccountConvention.DEBIT}
    }, INCOME{
        AccountConvention increase() {AccountConvention.CREDIT}
        AccountConvention decrease() {AccountConvention.DEBIT}
    }
}
```

#### 勘定の決まりごと

```groovy AccountConveintion.groovy
enum AccountConvention {
    DEBIT, CREDIT
}
```

この二つのクラスを書いていて、`Debit`とか`Credit`のクラスを生成するメソッドはこのあたりにあるといいなと思ってきた。

例えば、費用が増加した場合

```groovy
AccountChart.EXPENSE
        .increase()
        .by('売上原価', '商品が売れた')
        .for(item.asCredit())
```

という感じで書きたいですね。

そうすると、`AccountChart.groovy`を少し直さないといけないですね…


以下、宿題
---

#### Rearrange the following list of accounts and produce a trial balance.

+ Accounts Payable(900)
+ Accounts Receivable(1,400)
+ Capital(3,200)
+ Cash(2,000)
+ Drawing(400)
+ Equipment(1,800)
+ Fees income(2,600)
+ General expense(100)
+ Notes Payable(1,100)
+ Rent expense(500)
+ Salaries expense(800)
+ Supplies(600)
+ Supplies expense(200)


{% render_partial _includes/post/post_footer.html %}



