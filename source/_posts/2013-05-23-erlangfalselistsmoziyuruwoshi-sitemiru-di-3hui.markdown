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


<table>
<tbody>
<tr>
<td><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=4274067149" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
<td><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=4873114659" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
<td><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=1449309968" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
</tr>
<tr>
<td><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=B00B634R04" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
<td><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=B00AZOT4MG" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
<td><iframe src="http://rcm-jp.amazon.co.jp/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kkkjkrt-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=1933988789" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</td>
</tr>
</tbody>
</table>

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
> The module_info/0 function in each module returns a list of {Key,Value} tuples with information about the module. Currently, the list contain tuples with the following Keys: attributes, compile, exports, and imports. The order and number of tuples may change without prior notice.
> 
> ##### warning
> 
> The {imports,Value} tuple may be removed in a future release because Value is always an empty list. Do not write code that depends on it being present.
> 
> [参照先](http://erlang.org/doc/reference_manual/modules.html#id74571)

#### Explain

これはlistsモジュールでなく、すべてのモジュールに共通する関数ですな。
モジュールに関する情報を`{Key, Value}`のタプルリストで返します。現在のところ、

+ attributes
+ compile
+ exports
+ imports

といったキーです。なお、`{imports, Value}`タプルは常に空のリストしか返さないため、今後なくなる予定です。これに基づいたコードを記述しないで下さい。

#### Example

```erlang
1> lists:module_info().
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

たくさん出てきた…


### module_info/1

#### Erlang Document

> #### module_info
> 
> The call module_info(Key), where key is an atom, returns a single piece of information about the module.
> 
> [参照先](http://erlang.org/doc/reference_manual/modules.html#id74571)

#### Explain

`module_info`関数から`Key`を指定して取り出します。


#### Example

```erlang
1> lists:module_info(attributes).
[{vsn,[257948301539042745638557295194154171573]}]
```

`lists:module_info/0`の結果の一部が返ってきます。


### nth/2

#### Erlang Document

> #### nth(N, List) -> Elem
> 
> #### Types
> 
> + N = integer() >= 1
> + List = [T, ...]
> + Elem = T
> + T = term()
> 
> Returns the Nth element of List.
> 
> [参照先](http://erlang.org/doc/man/lists.html#nth-2)

#### Explain

`N`番目の要素を返します。


#### Example

```erlang
1> lists:nth(3,[a,2,c,4,e,5]).
c
```

3番目の要素`c`が返されます。


### nthtail/2

#### Erlang Document

> #### nthtail(N, List) -> Tail
> 
> #### Types
> 
> + N = integer() >= 0
> + List = [T, ...]
> + Elem = T
> + T = term()
> 
> Returns the Nth tail of List, that is, the sublist of List starting at N+1 and continuing up to the end of the list.
> 
> [参照先](http://erlang.org/doc/man/lists.html#nthtail-2)

#### Explain

`N+1`番から後ろの要素で作られるサブリストを返します。


#### Example

```erlang
1> lists:nthtail(3,[a,2,c,4,e,5]).
[4,e,5]
```

4番目から後ろの要素で構成されるリストが返されます。


### partition/2

#### Erlang Document

> #### partition(Pred, List) -> {Satisfying, NotSatisfying}
> 
> #### Types
> 
> + Pred = fun((Elem :: T) -> boolean())
> + List = Satisfying = NotSatisfying = [T]
> + T = term()
> 
> Partitions List into two lists, where the first list contains all elements for which Pred(Elem) returns true, and the second list contains all elements for which Pred(Elem) returns false.
> 
> [参照先](http://erlang.org/doc/man/lists.html#partition-2)

#### Explain

条件と分割対象リストを引数に取り、二つの新しいリストに分割します。最初のリストには条件を満たすものを、次のリストには条件を満たさないものを返します。


#### Example

```erlang
1> lists:partition(fun(X) -> X rem 2 =:= 0 end, [0,1,2,3,4,5,6,7]).
{[0,2,4,6],[1,3,5,7]}
```

偶数かどうか判定する条件とリストを与えています。
先頭のリストには条件を満たすもの(偶数)のリスト、後ろ側のリストには条件を満たさないもの(奇数)のリストが返されます。


### prefix/2

#### Erlang Document

> #### prefix(List1, List2) -> boolean()
> 
> #### Types
> 
> + List1 = List2 = [T]
> + T = term()
> 
> Returns true if List1 is a prefix of List2, otherwise false.
> 
> [参照先](http://erlang.org/doc/man/lists.html#prefix-2)

#### Explain

`List1`が`List2`の先頭部分である場合に`true`を、異なる場合には`false`を返します。


#### Example

```erlang
1> lists:prefix([1,2,3,4], [1,2,3,4,5,6,7]).
true
2> lists:prefix([a,b,c,d], [a,z,b,y,c,x]).
false
3> lists:prefix("co", "coexistence").
true
```

最初の例では、第一引数が第二引数の先頭部分と一致するので`true`が返されます。

次の例では、第一引数が第二引数の先頭部分と異なるので`false`が返されます。

最後の例では`"co"`は`"coexistence"`の最初の文字列部分と同じなので、`true`が返されます。


### reverse/1

#### Erlang Document

> #### reverse(List1) -> List2
> 
> #### Types
> 
> + List1 = List2 = [T]
> + T = term()
> 
> Returns a list with the elements in List1 in reverse order.
> 
> [参照先](http://erlang.org/doc/man/lists.html#reverse-1)

#### Explain

リストを逆順にして返します。


#### Example

```erlang
1> lists:reverse([9,8,7,6,5]).
[5,6,7,8,9]
```

リストの順番が逆になって返されています。


### reverse/2

#### Erlang Document

> #### reverse(List1, Tail) -> List2
> 
> #### Types
> 
> + List1 = [T]
> + Tail = term()
> + List2 = [T]
> + T = term()
> 
> Returns a list with the elements in List1 in reverse order, with the tail Tail appended.
> 
> [参照先](http://erlang.org/doc/man/lists.html#reverse-2)

#### Explain

リストを逆順にした上で、第二引数を追加した新しいリストを返します。


#### Example

```erlang
1> lists:reverse([5,4,3,2,1],[0]).
[1,2,3,4,5,0]
2> lists:reverse([5,4,3,2,1],[a,b,c]).
[1,2,3,4,5,a,b,c]
3> lists:reverse([5,4,3,2,1],0).
[1,2,3,4,5|0]
```

最初の例、および二番目の例では最初のリストが逆順になされた上で、次のリストが連結された新しいリストが返ってきます。

最後の例では、結果がリストでなくなります。

公式ドキュメントが間違っているっぽいですね…


### rkeymerge/3

#### Erlang Document

なかった＼(^o^)／


#### Example

```erlang
1> lists:rkeymerge(2,          
1>   [{a, 4}, {a, 3}, {a, 1}], 
1>   [{b, 5}, {b, 2}, {b, 0}]).
[{b,5},{a,4},{a,3},{b,2},{a,1},{b,0}]
2> lists:rkeymerge(2, 
2>   [{a, 1}, {a, 3}, {a, 4}],
2>   [{b, 0}, {b, 2}, {b, 5}]).
[{a,1},{a,3},{a,4},{b,0},{b,2},{b,5}]
3> lists:rkeymerge(2,          
3>   [{a, 4}, {a, 3}, {a, 2}, {a, 1}],
3>   [{b, 5}, {b, 2}, {b, 0}]).       
[{b,5},{a,4},{a,3},{b,2},{a,2},{a,1},{b,0}]
```

`lists:keymerge/3`のreverse版のようです。

+ 引数のリストは逆順にソートされている必要があります。
+ 同一のキーを持つ場合、第二引数のリストの要素が優先されます。

最初の例では、タプルの２番目の値をキーに逆順ソートされたリストが返されます。

次の例では、逆順にソートされていないため、マージがうまくなされていません。

最後の例では、同じキーがある場合の挙動を確認しています。

同じキー値をもつ要素(ここでは、`{a,2}`と`{b,2}`)がありますが、

`{b, 2}`の方が優先されているのがわかります。


### rmerge/2

#### Erlang Document

なかった＼(^o^)／


#### Example

```erlang
1>
```




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


