---
layout: post
title: "Erlangのlistsモジュールを試してみる - 第2回"
date: 2013-05-14 14:01
comments: true
categories: erlang
---
Erlangのlistsモジュールを試してみるの第二回は次のコマンドの結果について試していきます。


```erlang
lists:sublist(
    lists:sort(
        lists:map(
            fun({F, A}) ->
                atom_to_list(F) ++
                "/" ++
                integer_to_list(A)
            end,
            lists:module_info(exports))),
    17,
    16).
```

結果

```
[
  "keydelete/3",
  "keyfind/3",
  "keymap/3",
  "keymember/3",
  "keymerge/3",
  "keyreplace/4",
  "keysearch/3",
  "keysort/2",
  "keystore/4",
  "keytake/3",
  "last/1",
  "map/2",
  "mapfoldl/3",
  "mapfoldr/3",
  "max/1",
  "member/2"
]
```

です。



### keydelete/3

#### Erlang公式ドキュメント

> #### keydelete(Key, N, TupleList1) -> TupleList2
> 
> #### Types
> 
> + Key = term()
> + N = integer() >= 1 (1..tuple_size(Tuple))
> + TupleList1 = TupleList2 = [Tuple]
> + Tuple = tuple()
> 
> Returns a copy of TupleList1 where the first occurrence of a tuple whose Nth element compares equal to Key is deleted, if there is such a tuple.
> 
> [参照元](http://erlang.org/doc/man/lists.html#keydelete-3)

#### Explain

引数に与えられたタプルのリストのコピーを返します。ただし、リスト中のタプルで、タプルのN番目の要素が引数Keyと同じ値のもので最も先頭に近いものは取り除かれます。


#### Example

```
1> lists:keydelete(jim, 1,
1> [
1>     {mike, 32, programmer}, {jim, 22, engineer}, {tim, 40, scientist},
1>     {bob, 36, officer}, {lod, 22, dentist}, {iwan, 27, evangelist},
1>     {simon, 36, typist}, {dag, 34, student}, {jack, 21, programmer},
1>     {john, 21, engineer}, {mike, 27, scientist}, {jim, 24, officer}
1> ]).
[{mike,32,programmer},
 {tim,40,scientist},
 {bob,36,officer},
 {lod,22,dentist},
 {iwan,27,evangelist},
 {simon,36,typist},
 {dag,34,student},
 {jack,21,programmer},
 {john,21,engineer},
 {mike,27,scientist},
 {jim,24,officer}]
2> lists:keydelete(madscientist, 3,
2> [
2>     {mike, 32, programmer}, {jim, 22, engineer}, {tim, 40, scientist},
2>     {bob, 36, officer}, {lod, 22, dentist}, {iwan, 27, evangelist},
2>     {simon, 36, typist}, {dag, 34, student}, {jack, 21, programmer},
2>     {john, 21, engineer}, {mike, 27, scientist}, {jim, 24, officer}
2> ]).
[{mike,32,programmer},
 {jim,22,engineer},
 {tim,40,scientist},
 {bob,36,officer},
 {lod,22,dentist},
 {iwan,27,evangelist},
 {simon,36,typist},
 {dag,34,student},
 {jack,21,programmer},
 {john,21,engineer},
 {mike,27,scientist},
 {jim,24,officer}]
3> lists:keydelete(madscientist, 4,                                      
3> [                                                                     
3>     {mike, 32, programmer}, {jim, 22, engineer}, {tim, 40, scientist},
3>     {bob, 36, officer}, {lod, 22, dentist}, {iwan, 27, evangelist},   
3>     {john, 21, engineer}, {mike, 27, scientist}, {jim, 24, officer}   
3> ]).
[{mike,32,programmer},
 {jim,22,engineer},
 {tim,40,scientist},
 {bob,36,officer},
 {lod,22,dentist},
 {iwan,27,evangelist},
 {john,21,engineer},
 {mike,27,scientist},
 {jim,24,officer}]
```

まず最初の例では、リストの中で一番最初に出てくるキーの位置`1`番目の値が`jim`であるタプルを削除したリストが返されます。元のリストで9番目のタプルも同様の条件ですが、こちらは削除されません。

次の例ではリストで一致しない条件で実行しています。返されるリストは何も削除されていません。

最後の例ではリストのタプルの要素数より多い番号を指定して削除を実施します。もちろん、返されるリストは何も削除されていません。


### keyfind/3

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#keyfind-3)

#### Explain



#### Example

```
```


### keymap/3

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#keymap-3)

#### Explain



#### Example

```
```



### keymember/3

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#keymember-3)

#### Explain



#### Example

```
```



### keymerge/3

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#keymerge-3)

#### Explain



#### Example

```
```



### keyreplace/4

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#keyreplace-4)

#### Explain



#### Example

```
```



### keysearch/3

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#keysearch-3)

#### Explain



#### Example

```
```



### keysort/2

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#keysort-2)

#### Explain



#### Example

```
```



### keystore/4

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#keystore-4)

#### Explain



#### Example

```
```



### keytake/3

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#keytake-3)

#### Explain



#### Example

```
```



### last/1

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#last-1)

#### Explain



#### Example

```
```



### map/2

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#map-2)

#### Explain



#### Example

```
```



### mapfoldl/3

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#mapfoldl-3)

#### Explain



#### Example

```
```



### mapfoldr/3

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#mapfoldr-3)

#### Explain



#### Example

```
```



### max/1

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#max-1)

#### Explain



#### Example

```
```



### member/2

#### Erlang公式ドキュメント

> #### 
> 
> #### Types
> 
> + 
> + 
> 
> 
> [参照元](http://erlang.org/doc/man/lists.html#member-2)

#### Explain



#### Example

```
```



次回
----

次回は次の式の結果をやっていきます。

```erlang
lists:sublist(
    lists:sort(
        lists:map(
            fun({F, A}) ->
                atom_to_list(F) ++
                "/" ++
                integer_to_list(A)
            end,
            lists:module_info(exports))),
    17,
    16).
```



{% render_partial _includes/post/post_footer.html %}


