---
layout: post
title: "Erlangのリスト系モジュールを試してみる - 第0回"
date: 2013-05-11 01:50
comments: true
categories: erlang
---

こんにちわ、みけです。

Erlangの初心者・初級・初歩レベルの僕にとって、

まず色々とBIFだとか、基本的なモジュールを使いこなせるのが、

上達への近道かと思ったので、

`lists`モジュールについて、

すべての関数を試してみようと思います。


### モジュール情報を表示する

まず、モジュールの情報を表示するモジュールcのコマンドは

`m(Module)`関数です。

早速`lists`モジュールに試して見る。

```
1> m(lists).
Module lists compiled: Date: February 25 2013, Time: 19.27
Compiler options:  [{outdir,"/net/isildur/ldisk/daily_build/r16b_prebuild_master-opu_o.2013-02-25_20/otp_src_R16B/lib/stdlib/src/../ebin"},
                    {i,"/net/isildur/ldisk/daily_build/r16b_prebuild_master-opu_o.2013-02-25_20/otp_src_R16B/lib/stdlib/src/../include"},
                    {i,"/net/isildur/ldisk/daily_build/r16b_prebuild_master-opu_o.2013-02-25_20/otp_src_R16B/lib/stdlib/src/../../kernel/include"},
                    warnings_as_errors,debug_info]
Object file: /opt/local/lib/erlang/lib/stdlib-1.19.1/ebin/lists.beam
Exports: 
all/2                         nthtail/2
any/2                         partition/2
append/2                      prefix/2
append/1                      reverse/1
concat/1                      reverse/2
delete/2                      rkeymerge/3
dropwhile/2                   rmerge/2
duplicate/2                   rmerge/3
filter/2                      rmerge3/3
flatlength/1                  rukeymerge/3
flatmap/2                     rumerge/3
flatten/2                     rumerge/2
flatten/1                     rumerge3/3
foldl/3                       seq/2
foldr/3                       seq/3
foreach/2                     sort/2
keydelete/3                   sort/1
keyfind/3                     split/2
keymap/3                      splitwith/2
keymember/3                   sublist/3
keymerge/3                    sublist/2
keyreplace/4                  subtract/2
keysearch/3                   suffix/2
keysort/2                     sum/1
keystore/4                    takewhile/2
keytake/3                     ukeymerge/3
last/1                        ukeysort/2
map/2                         umerge/3
mapfoldl/3                    umerge/1
mapfoldr/3                    umerge/2
max/1                         umerge3/3
member/2                      unzip/1
merge/1                       unzip3/1
merge/2                       usort/2
merge/3                       usort/1
merge3/3                      zf/2
min/1                         zip/2
module_info/0                 zip3/3
module_info/1                 zipwith/3
nth/2                         zipwith3/4
ok

```

関数がたくさんあって、数えるの面倒いな…

### 詳細なモジュール情報を表示する

`lists:module_info/0`関数でも同じようなものを調べられる。

```
2> lists:module_info().
[{exports,[{append,2},
           {append,1},
           {subtract,2},
           {nth,2},
           {nthtail,2},
           {prefix,2},
           {suffix,2},
           {last,1},
           {seq,2},
           {seq,3},
           {sum,1},
           {duplicate,2},
           {min,1},
           {max,1},
           {sublist,3},
           {sublist,2},
           {zip,2},
           {unzip,1},
           {zip3,3},
           {unzip3,1},
           {zipwith,3},
           {zipwith3,4},
           {merge,1},
           {merge3,3},
           {rmerge3,...},
           {...}|...]},
 {imports,[]},
 {attributes,[{vsn,[257948301539042745638557295194154171573]}]},
 {compile,[{options,[{outdir,"/net/isildur/ldisk/daily_build/r16b_prebuild_master-opu_o.2013-02-25_20/otp_src_R16B/lib/stdlib/src/../ebin"},
                     {i,"/net/isildur/ldisk/daily_build/r16b_prebuild_master-opu_o.2013-02-25_20/otp_src_R16B/lib/stdlib/src/../include"},
                     {i,"/net/isildur/ldisk/daily_build/r16b_prebuild_master-opu_o.2013-02-25_20/otp_src_R16B/lib/stdlib/src/../../kernel/include"},
                     warnings_as_errors,debug_info]},
           {version,"4.9"},
           {time,{2013,2,25,19,27,47}},
           {source,"/net/isildur/ldisk/daily_build/r16b_prebuild_master-opu_o.2013-02-25_20/otp_src_R16B/lib/stdlib/src/lists.erl"}]}]
```

`exports`されている関数、

途中で表示が切れてまんがな…

### BIFとかリスト内包表記で頑張ってみる

今はエントリーの長さに影響をする関数の数を知りたいので、

リストの内包表記で`lists:module_info/0`の

結果を絞って、BIFの

+ `length/1` - リストの長さを取得する
+ `hd/1` - リストの先頭要素を取得する

を用いて`lists`モジュールの関数の数を数えてみる。

```
3> length(hd([Items || {Atom, Items} <- lists:module_info(), Atom =:= exports])).
80
```

というわけで、80もあるので、

16個ずつ調べていこうと思う。

> 追記 (2013/05/13 9:16)
> 
> 2013/05/12 21:19 頃に
> 
> [voluntasさん](https://github.com/voluntas)([@voluntas](http://twitter.com/voluntas))から
> メッセージをいただきました。
> 
> `length(lists:module_info(exports))`
> 
> で同じ事が取得できるとのこと。
> 
> 実際にやってみたら
> 
> ```
> 1> length(lists:module_info(exports)).
> 80
> ```
> 
> あ、できた、ありがとうございます！
> 
> ちなみにドキュメントにもありました。
> 
> [The module_info/0 and module_info/1 functions](http://erlang.org/doc/reference_manual/modules.html#id74571)
> 
> ドキュメント読まんとイケませんね
> 









### まとめ

まとめもクソもないのですが、

次回から約5回にわたって、

`lists`モジュールの関数を試していきます。


{% render_partial _includes/post/post_footer.html %}

