---
layout: post
title: "Erlangのlistsモジュールを試してみる - 第1回"
date: 2013-05-14 03:15
comments: true
categories: erlang
---

みけです。

Erlangのlistsモジュールを試してみるの第一回になります。

なお、最初の16個は

```erlang
ists:sublist(
    lists:sort(
        lists:map(
            fun({F, A}) ->
                atom_to_list(F) ++
                "/" ++
                integer_to_list(A)
            end,
            lists:module_info(exports))),
    1,
    16).
```

の実行結果


```
[
  "all/2",
  "any/2",
  "append/1",
  "append/2",
  "concat/1",
  "delete/2",
  "dropwhile/2",
  "duplicate/2",
  "filter/2",
  "flatlength/1",
  "flatmap/2",
  "flatten/1",
  "flatten/2",
  "foldl/3",
  "foldr/3",
  "foreach/2"
]
```

です。

### all/2 

#### Erlang公式ドキュメント

> #### all(Pred, List) -> boolean()
> 
> ##### Types
> 
> + Pred = fun((Elem :: T) -> boolean())
> + List = [T]
> + T = term()
> 
> Returns true if Pred(Elem) returns true for all elements Elem in List, otherwise false.
> 
> [参照元](http://erlang.org/doc/man/lists.html#all-2)


#### 解説

+ すべての要素が第一引数に与えられた関数で`true`になれば`true`を返します。
+ 第一引数に与えられた関数で`false`になるものが一つでもあれば、`false`を返します。


#### 実例

```
1> lists:all(fun(X) -> X rem 2 =:= 0 end, [0,2,4,6,8]).
true
2> lists:all(fun(X) -> X rem 2 =:= 0 end, [0,1,2,3,4]).
false
```

偶数であることを判断する関数を第一引数に与えています。

最初の例では全部偶数である`[0, 2, 4, 8]`が与えられています。

その結果、`true`が返ってきます。

一方、二番目の例では単なる1間隔の順列`[1, 2, 3, 4]`が与えられています。

したがって、`false`が返ってきます。


### any/2 

#### Erlang公式ドキュメント

> #### any(Pred, List) -> boolean()
> 
> ##### Types
> 
> + Pred = fun((Elem :: T) -> boolean())
> + List = [T]
> + T = term()
> 
> Returns true if Pred(Elem) returns true for at least one element Elem in List.
> 
> [参照元](http://erlang.org/doc/man/lists.html#any-2)


#### 解説

+ リストの要素のうち、一つでも第一引数に渡した関数が`true`を返せば`true`が返ってきます。
+ リストの要素すべてで、第一引数の関数を満たさなければ`false`が返ってきます。


#### 実例

```
1> lists:any(fun(X) -> is_integer(X) end,                                         1>     [0.0, a, "hoge", 1, fun(X) -> true end]).
true
2> lists:any(fun(X) -> is_integer(X) end,       
2>     [0.0, a, "hoge", [1], fun(X) -> true end]).
false
```

整数であることを判断する関数を第一引数として与えています。

最初の例ではリストの4番目の要素`1`が整数ですので、この関数から`true`が返ってきます。したがって、実行結果は`true`となります。

２つ目の例では先ほどは整数だった`1`をリスト`[1]`に変更しています。そのため、関数が`true`を返すものがありません。したがって、実行結果は`false`になります。


### append/1 

#### Erlang公式ドキュメント

> #### append(ListOfLists) -> List1
> 
> ##### Types
> 
> + ListOfLists = [List]
> + List = List1 = [T]
> + T = term()
> 
> Returns a list in which all the sub-lists of ListOfLists have been appended.
> 
> [参照元](http://erlang.org/doc/man/lists.html#append-1)


#### 解説

リストの中に含まれる複数のリストを結合した新しいリストを返します。


#### 実例

```
1> lists:append([[1], [2,3,4],[a,b,c]]).
[1,2,3,4,a,b,c]
2> lists:append([[1],[2,[3,4],5],[a,b,c]]).
[1,2,[3,4],5,a,b,c]
```

最初の例ではリストの中にある複数のリストが結合されて一つのリストにまとまります。

次の例ではリストの中にある複数のリストが結合されて一つのリストにまとまっていますが、リスト中のリスト中のリストは展開されません。


### append/2 

#### Erlang公式ドキュメント

> #### append(List1, List2) -> List3
> 
> #### Types
> 
> + List1 = List2 = List3 = [T]
> + T = term()
> 
> Returns a new list List3 which is made from the elements of List1 followed by the elements of List2.
> 
> [参照元](http://erlang.org/doc/man/lists.html#append-2)


#### 解説

リストを結合して新しいリストを返します。


#### 実例

```
1> lists:append("ir", "of").
"irof"
```

リスト`"ir"`とリスト`"of"`が結合されて、新しいリスト`"irof"`が返されます。

### concat/1 

#### Erlang公式ドキュメント

> #### concat(Things) -> string()
> 
> #### Types
> 
> + Things = [Thing]
> + Thing = atom() | integer() | float() | string()
> 
> Concatenates the text representation of the elements of Things. The elements of Things can be atoms, integers, floats or strings.
> 
> [参照元](http://erlang.org/doc/man/lists.html#concat-1)


#### 解説

リスト中の要素の文字列表現を結合する。要素はアトム、整数、浮動小数点数、文字列。


#### 実例

```
1> lists:concat([lists, ':', "concat", '/', 1]).
"lists:concat/1"
```

アトム、アトム、文字列、アトム、整数を結合して文字列を出力しています。

### delete/2 

#### Erlang公式ドキュメント

> #### delete(Elem, List1) -> List2
> 
> #### Types
> 
> + Elem = T
> + List1 = List2 = [T]
> + T = term()
> 
> Returns a copy of List1 where the first element matching Elem is deleted, if there is such an element.
> 
> [参照元](http://erlang.org/doc/man/lists.html#delete-2)


#### 解説

リストから指定要素にマッチする最初の要素を削除したリストを返します。


#### 実例

```
1> lists:delete(a, [q,a,w,s,e]).
[q,w,s,e]
2> lists:delete(a, [f,o,o,b,a,r,b,a,z]).
[f,o,o,b,r,b,a,z]
3> lists:delete(hoge, [foo, bar, baz]).
[foo,bar,baz]
```

最初の例では、リストの中に指定した要素`a`があるため、それを削除したリストが返されます。

次の例では、リストの中に指定した要素`a`が２つありますが、最初の`a`が削除されたリストが返されます。

最後の例では、リストの中に指定した要素`hoge`と同じ要素がないため、元のリストと同じ内容のリストが返されます。


### dropwhile/2 

#### Erlang公式ドキュメント

> #### dropwhile(Pred, List1) -> List2
> 
> #### Types
> 
> + Pred = fun((Elem :: T) -> boolean())
> + List1 = List2 = [T]
> + T = term()
> 
> Drops elements Elem from List1 while Pred(Elem) returns true and returns the remaining list.
> 
> [参照元](http://erlang.org/doc/man/lists.html#dropwhile-2)


#### 解説

引数として与えた関数`Pred(Elem)`が`true`を返す間、要素`Elem`をリストから削除して、残ったリストを返します。


#### 実例

```
1> lists:dropwhile(fun(X) -> X < 5 end, [1,2,3,4,5,6,7]).
[5,6,7]
2> lists:dropwhile(fun(X) -> X > 2 end,                       
2>     [4,4,5,4,2,2,1,3,3,5]). 
[2,2,1,3,3,5]
```

最初の例では要素が`5`より小さい要素は除外され、`5`より大きいリストが返されます。

次の例では、要素が`2`より大きい間の要素は除外され、初めて`2`が出てきた後の要素が残ったリストが返されます。(返されたリストには関数が`true`を返すものが含まれます)

### duplicate/2 

#### Erlang公式ドキュメント

> #### duplicate(N, Elem) -> List
> 
> #### Types
> 
> + N = integer() >= 0
> + Elem = T
> + List = [T]
> + T = term()
> 
> Returns a list which contains N copies of the term Elem.
> 
> [参照元](http://erlang.org/doc/man/lists.html#duplicate-2)


#### 解説

要素`Elem`を`N`回繰り返したリストが返されます。


#### 実例

```
1> lists:duplicate(5, irof).
[irof,irof,irof,irof,irof]
```

要素`irof`が`5`回繰り返されたリストが返されます。


### filter/2 

#### Erlang公式ドキュメント

> #### filter(Pred, List1) -> List2
> 
> #### Types
> 
> + Pred = fun((Elem :: T) -> boolean())
> + List1 = List2 = [T]
> + T = term()
> 
> List2 is a list of all elements Elem in List1 for which Pred(Elem) returns true.
> 
> [参照元](http://erlang.org/doc/man/lists.html#filter-2)


#### 解説

リストの中から条件を満たす要素を取り出したリストを返します。


#### 実例

```
1> lists:filter(fun(X) -> X rem 3 =:= 0 end,
1>     [1,2,3,4,5,6,7,8]).                  
[3,6]
```

リストの中から`3`の剰余が`0`のものを取り出したリストを返します。


### flatlength/1 

#### Erlang公式ドキュメント

> #### flatlength(DeepList) -> integer() >= 0
> 
> #### Types
> 
> + DeepList = [term() | DeepList]
> 
> Equivalent to `length(flatten(DeepList))`, but more efficient.
> 
> [参照元](http://erlang.org/doc/man/lists.html#flatlength-1)


#### 解説

BIFの`length/1`関数と同様な関数ですが、より便利です。


#### 実例

```
1> lists:flatlength([[1,2,3],4,[[5,6],7]]).
7
2> length([[1,2,3],4,[[5,6],7]]).
3
```

内部にリストを含むリストに対して内部のリストの要素数を含む要素数を返します。最初の例では内部リストの要素の数を含めて`7`を返します。

一方、BIFの`length/1`では、内部リストを一つの要素として数えるため、`3`が返ってきます。

### flatmap/2 

#### Erlang公式ドキュメント

> #### flatmap(Fun, List1) -> List2
> 
> #### Types
> 
> + Fun = fun((A) -> [B])
> + List1 = [A]
> + List2 = [B]
> + A = B = term()
> 
> Takes a function from As to lists of Bs, and a list of As (List1) and produces a list of Bs by applying the function to every element in List1 and appending the resulting lists.
> 
> [参照元](http://erlang.org/doc/man/lists.html#flatmap-2)

#### 解説

第一引数には要素Aからリスト[B]を生成する関数を取ります。第二引数はリストです。そしてリスト[B]が結合されたリストが返されます。


#### 実例

```
1> lists:flatmap(fun(X) -> [X * 2, X * 2 + 1] end,            
1>     [0,1,2,3]).
[0,1,2,3,4,5,6,7]
2> lists:flatmap(fun(X) -> X * X end, [1,2,3,4]).
** exception error: bad argument
     in operator  ++/2
        called as 16 ++ []
     in call from lists:flatmap/2 (lists.erl, line 1235)
     in call from lists:flatmap/2 (lists.erl, line 1235)
```

最初の例での第一引数の関数は、入力された要素を倍にした値と、倍にした値に1を加えた値の二つを要素としてもつリストを返します。これをリスト`[0, 1, 2, 3]`に対して実行し、最終的にそれらを結合したリスト`[0,1,2,3,4,5,6,7]`が返ってきます。

次の例では、第一引数の関数は入力された要素の二乗を返します。これをリスト`[1, 2, 3, 4]`に対して実行します。しかし第一引数が返す値はリストではないため、例外が発生して終了します。


### flatten/1 

#### Erlang公式ドキュメント

> #### flatten(DeepList) -> List
> 
> #### Types
> 
> + DeepList = [term() | DeepList]
> + List = [term()]
> 
> Returns a flattened version of DeepList.
> 
> [参照元](http://erlang.org/doc/man/lists.html#flatten-1)

#### 解説

内部にリストを持つリストのネストをなくしたリストを返します。


#### 実例

```
1> lists:flatten([1,2,[3,4,[5],6],[[[7],8]],9]).
[1,2,3,4,5,6,7,8,9]
```

内部にリストを持つリストから、ネストなしリストを返します。


### flatten/2 

#### Erlang公式ドキュメント

> #### flatten(DeepList, Tail) -> List
> 
> #### Types
> 
> + DeepList = [term() | DeepList]
> + Tail = List = [term()]
> 
> Returns a flattened version of DeepList with the tail Tail appended.
> 
> [参照元](http://erlang.org/doc/man/lists.html#flatten-2)

#### 解説

最初のネストがあるリストに`lists:flatten/1`を施して、第二引数のリストを追加したリストを返します。


#### 実例

```
1> lists:flatten(                         
1>     [1,[2,3,[[4],5]],6], [7,8]).
[1,2,3,4,5,6,7,8]
2> lists:flatten(                    
2>     [1,[2,3,[[4],5]],6], [[7],8]).
[1,2,3,4,5,6,[7],8]
```

最初の例では、ネストしたリストからネストを外し、第二引数のリストを追加したリストが返ってきます。

次の例でも、第一引数のネストしたリストのネストが外れ、第二引数のリストを追加したものが返ってきますが、第二引数のリストのネストは残ったままとなります。


### foldl/3 

#### Erlang公式ドキュメント

> #### foldl(Fun, Acc0, List) -> Acc1
> 
> #### Types
> 
> + Fun = fun((Elem :: T, AccIn) -> AccOut)
> + Acc0 = Acc1 = AccIn = AccOut = term()
> + List = [T]
> + T = term()
> 
> Calls Fun(Elem, AccIn) on successive elements A of List, starting with AccIn == Acc0. Fun/2 must return a new accumulator which is passed to the next call. The function returns the final value of the accumulator. Acc0 is returned if the list is empty.
> 
> [参照元](http://erlang.org/doc/man/lists.html#foldl-3)

#### 解説

リストの左側から１つずつ要素を取り出し、第一引数の関数を連続で呼び出します。第一引数の関数は二つの引数を取り、次回の引数として使われる値を返します。第二引数には初期値を与えます。`lists:foldl`関数は最後の第一引数の関数の結果を返します。


#### 実例

```
1> lists:foldl(fun(X, Accum) -> X + Accum end,
1>     0, [1,2,3,4,5]).
15
2> lists:foldl(fun(X, Accum) -> [X * X | Accum] end,
2>     [], [1,2,3,4,5]).                            
[25,16,9,4,1]
```

最初の例では初期値0からリスト`[1,2,3,4,5]`を順番に足していき、最後の結果`15`を返します。

次の例では、初期値`[]`のリストにリスト`[1,2,3,4,5]`の要素の二乗を追加していき、最終的にリスト`[25,16,9,4,1]`を返します。


### foldr/3 

#### Erlang公式ドキュメント

> #### foldr(Fun, Acc0, List) -> Acc1
> 
> #### Types
> 
> + Fun = fun((Elem :: T, AccIn) -> AccOut)
> + Acc0 = Acc1 = AccIn = AccOut = term()
> + List = [T]
> + T = term()
> 
> Like `foldl/3`, but the list is traversed from right to left.
> 
> [参照元](http://erlang.org/doc/man/lists.html#foldr-3)

#### 解説

`lists:foldl/3`と同様に順次計算していきますが、`lists:foldr/3`では関数の引数をリストの右から左の順番で取っていきます。


#### 実例

```
1> lists:foldr(fun(X, Accum) -> X + Accum end,
1>     0, [1,2,3,4,5]).
15
2> lists:foldr(fun(X, Accum) -> [{X, X * X}|Accum] end,
2>     [], [1,2,3,4,5]).
[{1,1},{2,4},{3,9},{4,16},{5,25}]
```

最初の例では、初期値`0`に対してリスト`[1,2,3,4,5]`の要素を右側から足していき、最後の結果`15`を返します。

次の例では、初期値`[]`に、リスト`[1,2,3,4,5]`の元の数とその二乗のペアを右側から追加していき、最後の結果`[{1,1}, {2,4}, {3,9}, {4,16}, {5,25}]`を返します。


### foreach/2 

#### Erlang公式ドキュメント

> #### foreach(Fun, List) -> ok
> 
> #### Types
> 
> + Fun = fun((Elem :: T) -> term())
> + List = [T]
> + T = term()
> 
> Calls Fun(Elem) for each element Elem in List. This function is used for its side effects and the evaluation order is defined to be the same as the order of the elements in the list.
> 
> [参照元](http://erlang.org/doc/man/lists.html#foreach-2)


#### 解説

第一引数に与えられた関数をリストの要素すべてに適用します。この関数は副作用のある処理に用いられます。処理順はリストの順番に実行されます。


#### 実例

```
3> lists:foreach(fun(X) ->                                                
3>     io:format("~p x ~p = ~p~n", 
3>         [X, X, X * X]) end,
3>     [1,2,3,4,5]).
1 x 1 = 1
2 x 2 = 4
3 x 3 = 9
4 x 4 = 16
5 x 5 = 25
ok
```

リストの要素を順番に標準出力に出力しています。


次回
----

次回は次の式の結果をやっていきます。

```erlang
ists:sublist(
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

