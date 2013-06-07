---
layout: post
title: "Groovyでpegdownをさわってみる"
date: 2013-06-08 05:25
comments: true
categories: [groovy, java]
---

みけです。

<img src="//groovy.codehaus.org/images/groovy-logo-medium.png" style="width : 450px;"/>

前回に引き続き、Javaのmarkdownパーサーを試してみます。

で、今回は

[pegdown](https://github.com/sirthias/pegdown)
===

を試してみました。

サンプルコードは前回とほとんど変わりません。

```groovy pegdown-sample.groovy
@Grab('org.pegdown:pegdown:1.2.1')

def processor = new org.pegdown.PegDownProcessor()

def original = $/
This is Top Header
---

This is second Header
===

### This is topic

#### Lists Item

${(1..3).collect {"+ item${it}"}.join('\n')}

and

${(1..3).collect {"1. item${it}"}.join('\n')}

#### Links

+ [mike-neck's site](http://mike-neck.github.io/)
+ [mike-neck's dq site](http://mikeneckdq.blog.fc2.com/)
+ ![groovy image](http://groovy.codehaus.org/images/groovy-logo-medium.png)

#### Html Tags

<img src='//googledrive.com/host/0B4hhdHWLP7RRdHRGZ3ZrZU90Q00' style='width : 400px;'>

#### Codes

function `lists:reverse/1` returns a List.

tag `<em>` means emphasis

``groovy
def string = 'This is groovy code.'
``

#### emphasis

*em?*

**bold?**

#### Blockquotes

> This is a blockquotes
> from here.

/$

processor.markdownToHtml(original)
```

実行結果はこんな感じ。

```html
<h2>This is Top Header</h2><h1>This is second Header</h1><h3>This is topic</h3><h4>Lists Item</h4>
<ul>
  <li>item1</li>
  <li>item2</li>
  <li>item3</li>
</ul><p>and</p>
<ol>
  <li>item1</li>
  <li>item2</li>
  <li>item3</li>
</ol><h4>Links</h4>
<ul>
  <li><a href="http://mike-neck.github.io/">mike-neck's site</a></li>
  <li><a href="http://mikeneckdq.blog.fc2.com/">mike-neck's dq site</a></li>
  <li><img src="http://groovy.codehaus.org/images/groovy-logo-medium.png"  alt="groovy image"/></li>
</ul><h4>Html Tags</h4><p><img src='//googledrive.com/host/0B4hhdHWLP7RRdHRGZ3ZrZU90Q00' style='width : 400px;'></p><h4>Codes</h4><p>function <code>lists:reverse/1</code> returns a List.</p><p>tag <code>&lt;em&gt;</code> means emphasis</p><p><code>groovy
def string = &#39;This is groovy code.&#39;
</code></p><h4>emphasis</h4><p><em>em?</em></p><p><strong>bold?</strong></p><h4>Blockquotes</h4>
<blockquote><p>This is a blockquotes from here.</p>
</blockquote>
```

markdownjとの比較
---

markdownjではこんな不具合が有りました。

+ `===`がレンダーされない
+ 上の行にテキストがある状態で'---'がタグ`<tr/>`にレンダーされる
+ 当然ながら、GitHubっぽいコードスニペットはレンダーできない

一方、これらのうち、

+ `===`がレンダーされない
+ 上の行にテキストがある状態で'---'がタグ`<tr/>`にレンダーされる

はgepdownでは解消されているようです。

残念ながら、

+ 当然ながら、GitHubっぽいコードスニペットはレンダーできない

というのはあります。

というわけで、pegdownの方が、良さげな感じがします。


{% render_partial _includes/post/post_footer.html %}

