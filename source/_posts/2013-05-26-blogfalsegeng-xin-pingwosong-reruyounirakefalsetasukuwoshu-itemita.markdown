---
layout: post
title: "blogの更新pingを送れるようにrakeのタスクを書いてみた"
date: 2013-05-26 21:38
comments: true
categories: ruby
---

こんにちわ、みけです。

お金がないので、ブログを読んでもらって、

google adsenseをみんなに押してもらわないと、

死んでしまいます。


さて、そんなことはどうでもよくて、

bloggerでブログをやっていたときは、

googleさんが勝手に検索エンジンに乗せてくれるので、

検索しやすかったのですが、

githubでブログを書くようになって、

検索で出にくくなっていたので、

更新pingをrakeで送りつけるようにしました。

まあ、「更新ping ruby」でググった結果をrakeのタスクにしただけですが…

で、追加したのがこんな感じのタスクです。

```ruby rakefile
require "yaml"
require "xmlrpc/client"

#-- sending ping --#
desc "Sedning ping to Web Search Engines"
task :ping do
  site_config = YAML.load(IO.read('_config.yml'))
  blog_title = site_config['title']
  blog_url = site_config['url']
  ping_url = YAML.load(IO.read('ping.yml'))
  ping_url.each do |url|
    ping = XMLRPC::Client.new2(url)
    begin
      result = ping.call('weblogUpdates.ping', blog_title, blog_url)
      puts "#{url} : #{result}"
    rescue => e
      puts "#{url} : #{e}"
    end
  end
end
```

この新たに追加したタスクを後は、gen_deployタスクに追加します。

```ruby rakefile
desc "Generate website and deploy"
task :gen_deploy => [:integrate, :generate, :deploy, :ping] do
end
```


あと、適当に`ping.yml`に更新pingを送りつけるサイトを記述すればおｋです。

```yaml ping.yml
- http://blogsearch.google.com/ping/RPC2
- http://api.my.yahoo.co.jp/RPC2
- http://blog.goo.ne.jp/XMLRPC
- http://ping.bloggers.jp/rpc/
- http://ping.rss.drecom.jp/
- http://ping.fc2.com/
- http://rpc.weblogs.com/RPC2
- http://rpc.reader.livedoor.com/ping
- http://ping.blogranking.net/
- http://www.blogpeople.net/ping/
```

{% render_partial _includes/post/post_footer.html %}

