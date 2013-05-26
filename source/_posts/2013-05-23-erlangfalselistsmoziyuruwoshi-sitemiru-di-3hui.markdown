---
layout: post
title: "Erlangのlistsモジュールを試してみる - 第3回"
date: 2013-05-23 13:01
comments: true
categories: erlang
---

Erlangのlistsモジュールを試してみるの第三回は次のコマンドの結果について試していきます。


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
    33,
    16).
```

結果

```
[
  "merge/1",
  "merge/2",
  "merge/3",
  "merge3/3",
  "min/1",
  "module_info/0",
  "module_info/1",
  "nth/2",
  "nthtail/2",
  "partition/2",
  "prefix/2",
  "reverse/1",
  "reverse/2",
  "rkeymerge/3",
  "rmerge/2",
  "rmerge/3"
]
```


### merge/1

#### Erlang Document

> #### merge(ListOfLists) -> List1
> 
> #### Types
> 
> + ListOfLists = [List]
> + List = List1 = [T]
> + T = term()
> 
> Returns the sorted list formed by merging all the sub-lists of ListOfLists. All sub-lists must be sorted prior to evaluating this function. When two elements compare equal, the element from the sub-list with the lowest position in ListOfLists is picked before the other element.
> 
> [参照先](http://erlang.org/doc/man/lists.html#merge-1)

#### Explain

リスト内のリストをソートしてマージしたリストを返します。なお、リスト内のリストは事前にソートされている必要があります。


#### Example

```erlang
1> lists:merge([
1> [0,2,4],     
1> [3,6,9],     
1> [1,5,7]]).   
[0,1,2,3,4,5,6,7,9]
```

複数のリストがマージされて新しいリストが返されます。


### merge/2

#### Erlang Document

> #### merge(List1, List2) -> List3
> 
> #### Types
> 
> + List1 = [X]
> + List2 = [Y]
> + List3 = [(X | Y)]
> + X = Y = term()
> 
> Returns the sorted list formed by merging List1 and List2. Both List1 and List2 must be sorted prior to evaluating this function. When two elements compare equal, the element from List1 is picked before the element from List2.
> 
> [参照先](http://erlang.org/doc/man/lists.html#merge-2)

#### Explain

二つのリストをソートしてマージした新しいリストを返します。二つのリストは事前にソートされている必要があります。


#### Example

```erlang
1> lists:merge([1,3,4,6],[0,2,5,7]).
[0,1,2,3,4,5,6,7]
```

二つのリストがマージされた新しいリストが返されます。


### merge/3

#### Erlang Document

> #### merge(Fun, List1, List2) -> List3
> 
> #### Types
> 
> + Fun = fun((A, B) -> boolean())
> + List1 = [A]
> + List2 = [B]
> + List3 = [(A | B)]
> + A = B = term()
> 
> Returns the sorted list formed by merging List1 and List2. Both List1 and List2 must be sorted according to the [ordering function](http://www.erlang.org/doc/man/lists.html#ordering_function) `Fun` prior to evaluating this function. `Fun(A, B)` should return `true` if `A` compares less than or equal to `B` in the ordering, `false` otherwise. When two elements compare equal, the element from List1 is picked before the element from List2.
> 
> [参照先](http://erlang.org/doc/man/lists.html#merge-3)

#### Explain

`List1`と`List2`をソートしてマージした新しいリストを返します。引数のリストは事前にソートされている必要があります。`Fun(A, B)`はソートオーダーに関して`A`が先に来るべきである場合は`true`を、そうでない場合は`false`を返します。同じ値の要素がある場合は、`List1`のものが`List2`のものに優先されます。


#### Example

```erlang
1> lists:merge(fun(A, B) -> A > B end,[6,4,3,1],[7,5,2,0]).
[7,6,5,4,3,2,1,0]
2> lists:merge(fun({A,_}, {B,_}) -> A =< B end,            
2> [{1,a},{3,a},{4,a},{5,a}],                  
2> [{0,b},{2,b},{4,b},{6,b}]).                 
[{0,b},{1,a},{2,b},{3,a},{4,a},{4,b},{5,a},{6,b}]
```

最初の例では、降順にソートする関数で評価してリストをマージして新しいリストを返します。

後者の例では、昇順ソートです。同一の値の場合に`List1`から要素が取得されています。


### merge3/3

#### Erlang Document

> #### merge3(List1, List2, List3) -> List4
> 
> #### Types
> 
> + List1 = [X]
> + List2 = [Y]
> + List3 = [Z]
> + List4 = [(X | Y | Z)]
> + X = Y = Z = term()
> 
> Returns the sorted list formed by merging List1, List2 and List3. All of List1, List2 and List3 must be sorted prior to evaluating this function. When two elements compare equal, the element from List1, if there is such an element, is picked before the other element, otherwise the element from List2 is picked before the element from List3.
> 
> [参照先](http://erlang.org/doc/man/lists.html#merge3-3)

#### Explain

三つのリストをソートしてマージした新しいリストを返します。引数のリストは事前にソートされている必要があります。もし同じ値の要素があった場合は`List1`、`List2`、`List3`の順番で優先されます。


#### Example

```
50> lists:merge3(
50>   [2, 5, 8],
50>   [1, 4, 9],
50>   [3, 6, 7]).
[1,2,3,4,5,6,7,8,9]
```


### min/1

#### Erlang Document

> #### min(List) -> Min
> 
> #### Types
> 
> + List = [T, …]
> + Min = T
> + T = term()
> 
> Returns the first element of List that compares less than or equal to all other elements of List.
> 
> [参照先](http://erlang.org/doc/man/lists.html#min-1)

#### Explain

リスト中、最初に現れた最も小さい要素を返します。


#### Example

```erlang
1> lists:min([5, 3, 1, 4, 5]).
1
2> lists:min([h, b, n, t, l]).
b
3> lists:min([0,0.0]).
0
4> lists:min([0.0,0]).
0.0
5> lists:min(['A', 123, a, 123.0]).
123
```

最初の例では引数のリストの最小整数1が返されます。

次の例では引数のリストの最小のatom`b`が返されます。

三番目と四番目の例では整数と浮動小数点数の0を比較して最初に現れた要素が返されます。

最後の例では整数と浮動小数点数とatomを比較しています。最初に現れた最小の要素(整数)が返されます。


### module_info/0

#### Erlang Document

> #### module_info
> 
> #### Types
> 
> + 
> + 
> 
> 
> 
> [参照先](http://erlang.org/doc/man/lists.html#module_info-0)

#### Explain



#### Example

```erlang
1>
```




### module_info/1

#### Erlang Document

> #### module_info
> 
> #### Types
> 
> + 
> + 
> 
> 
> 
> [参照先](http://erlang.org/doc/man/lists.html#module_info-1)

#### Explain



#### Example

```erlang
1>
```




### nth/2

#### Erlang Document

> #### nth
> 
> #### Types
> 
> + 
> + 
> 
> 
> 
> [参照先](http://erlang.org/doc/man/lists.html#nth-2)

#### Explain



#### Example

```erlang
1>
```




### nthtail/2

#### Erlang Document

> #### nthtail
> 
> #### Types
> 
> + 
> + 
> 
> 
> 
> [参照先](http://erlang.org/doc/man/lists.html#nthtail-2)

#### Explain



#### Example

```erlang
1>
```




### partition/2

#### Erlang Document

> #### partition
> 
> #### Types
> 
> + 
> + 
> 
> 
> 
> [参照先](http://erlang.org/doc/man/lists.html#partition-2)

#### Explain



#### Example

```erlang
1>
```




### prefix/2

#### Erlang Document

> #### prefix
> 
> #### Types
> 
> + 
> + 
> 
> 
> 
> [参照先](http://erlang.org/doc/man/lists.html#prefix-2)

#### Explain



#### Example

```erlang
1>
```




### reverse/1

#### Erlang Document

> #### reverse
> 
> #### Types
> 
> + 
> + 
> 
> 
> 
> [参照先](http://erlang.org/doc/man/lists.html#reverse-1)

#### Explain



#### Example

```erlang
1>
```




### reverse/2

#### Erlang Document

> #### reverse
> 
> #### Types
> 
> + 
> + 
> 
> 
> 
> [参照先](http://erlang.org/doc/man/lists.html#reverse-2)

#### Explain



#### Example

```erlang
1>
```




### rkeymerge/3

#### Erlang Document

> #### rkeymerge
> 
> #### Types
> 
> + 
> + 
> 
> 
> 
> [参照先](http://erlang.org/doc/man/lists.html#rkeymerge-3)

#### Explain



#### Example

```erlang
1>
```




### rmerge/2

#### Erlang Document

> #### rmerge
> 
> #### Types
> 
> + 
> + 
> 
> 
> 
> [参照先](http://erlang.org/doc/man/lists.html#rmerge-2)

#### Explain



#### Example

```erlang
1>
```



{% render_partial _includes/post/post_footer.html %}


### rmerge/3

#### Erlang Document

> #### rmerge
> 
> #### Types
> 
> + 
> + 
> 
> 
> 
> [参照先](http://erlang.org/doc/man/lists.html#rmerge-3)

#### Explain



#### Example

```erlang
1>
```



{% render_partial _includes/post/post_footer.html %}

