---
layout: post
title: "Groovyでmarkdownjをさわってみる"
date: 2013-06-07 21:10
comments: true
categories: java, groovy
---

<img src="googledrive.com/host/0B4hhdHWLP7RRcURrSlZiNXlNVVk" style="width : 450px;"/>

こんにちわ、みけです。

このブログはOCTOPRESSで作っていますが、

どうしてもやりたい処理があったら、

rubyを勉強しないとアババな感じなので、

Javaでも同じようなものを作ってみたいと思っています。


OCTOPRESSっぽいツールに必要そうな機能
===

はこんな感じかなと思っています。

+ タスクを指定して実行する
+ git連携
+ previewできる
+ マークダウンをHTMLに変換する

これらのうち、最初のタスクを指定して実行するはGradleでやればよさそうです。

git連携はJGitを使えばできそうです。

previewはJettyだか、nettyだか、glassfishを使えばできそうです。

で、最後の

JavaでマークダウンをHTMLに変換する
===

はわからないので、

ちょっと試して見ることにしました。

まず、筆頭にくる **markdownj** を試してみることにしました

```groovy markdown-sample.groovy
@Grab('com.madgag:markdownj-core:0.4.1')

import com.petebevin.markdown.*

def proc = new MarkdownProcessor()

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

#### Html Tags

<img src='//googledrive.com/host/0B4hhdHWLP7RRdHRGZ3ZrZU90Q00' style='width : 400px;'>

#### Codes

function `lists:reverse/1` returns a List.

tag `<em>` means emphasis

#### emphasis

*em?*

**bold?**

#### Blockquotes

> This is a blockquotes
> from here.

/$

proc.markdown(original)
```

これの結果は次のようになります。

```html
<p>This is Top Header
<hr /></p>

<p>This is second Header
===</p>

<h3>This is topic</h3>

<h4>Lists Item</h4>

<ul>
<li>item1</li>
<li>item2</li>
<li>item3</li>
</ul>

<p>and</p>

<ol>
<li>item1</li>
<li>item2</li>
<li>item3</li>
</ol>

<h4>Links</h4>

<ul>
<li><a href="http://mike-neck.github.io/">mike-neck's site</a></li>
<li><a href="http://mikeneckdq.blog.fc2.com/">mike-neck's dq site</a></li>
</ul>

<h4>Html Tags</h4>

<p><img src='//googledrive.com/host/0B4hhdHWLP7RRdHRGZ3ZrZU90Q00' style='width : 400px;'></p>

<h4>Codes</h4>

<p>function <code>lists:reverse/1</code> returns a List.</p>

<p>tag <code>&lt;em&gt;</code> means emphasis</p>

<h4>emphasis</h4>

<p><em>em?</em></p>

<p><strong>bold?</strong></p>

<h4>Blockquotes</h4>

<blockquote>
  <p>This is a blockquotes
  from here.</p>
</blockquote>
```

今のところ、残念な点は

+ `===`がレンダーされない
+ 上の行にテキストがある状態で'---'がタグ`<tr/>`にレンダーされる
+ 当然ながら、GitHubっぽいコードスニペットはレンダーできない

ちょっと残念なので、

他のマークダウンパーサーも試してみようと思います。


{% render_partial _includes/post/post_footer.html %}

