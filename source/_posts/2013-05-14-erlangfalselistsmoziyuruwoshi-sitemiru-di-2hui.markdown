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

> #### keyfind(Key, N, TupleList) -> Tuple | false
> 
> #### Types
> 
> + Key = term()
> + N = integer() >= 1 (1..tuple_size(Tuple))
> + TupleList = [Tuple]
> + Tuple = tuple()
> 
> Searches the list of tuples TupleList for a tuple whose Nth element compares equal to Key. Returns Tuple if such a tuple is found, otherwise false.
> 
> [参照元](http://erlang.org/doc/man/lists.html#keyfind-3)

#### Explain

タプルのリストから指定位置の要素について`Key`に一致する最初のタプルを返します。


#### Example

```
1> lists:keyfind(36, 2,
1> [
1>     {mike, 32, programmer}, {jim, 22, engineer}, {tim, 40, scientist},
1>     {bob, 36, officer}, {lod, 22, dentist}, {iwan, 27, evangelist},
1>     {simon, 36, typist}, {dag, 34, student}, {jack, 21, programmer},
1>     {john, 21, engineer}, {mike, 27, scientist}, {jim, 24, officer}
1> ]).
{bob,36,officer}
2> lists:keyfind(jim, 1, 
2> [
2>     {mike, 32, programmer}, {jim, 22, engineer}, {tim, 40, scientist},
2>     {bob, 36, officer}, {lod, 22, dentist}, {iwan, 27, evangelist},
2>     {simon, 36, typist}, {dag, 34, student}, {jack, 21, programmer},
2>     {john, 21, engineer}, {mike, 27, scientist}, {jim, 24, officer}
2> ]).
{jim,22,engineer}
3> lists:keyfind(madscientist, 3,
3> [
3>     {mike, 32, programmer}, {jim, 22, engineer}, {tim, 40, scientist},
3>     {bob, 36, officer}, {lod, 22, dentist}, {iwan, 27, evangelist},
3>     {simon, 36, typist}, {dag, 34, student}, {jack, 21, programmer},
3>     {john, 21, engineer}, {mike, 27, scientist}, {jim, 24, officer}
3> ]).
false
```

最初の例ではタプルの二番目の値が`36`のものを探して、返します。

次の例ではタプルの最初の値が`jim`のものを探して返しますが、9番目に現れるものは返されません。

最後の例では一致するものがない条件で検索を行いますが、存在しないため`false`が返ってきます。


### keymap/3

#### Erlang公式ドキュメント

> #### keymap(Fun, N, TupleList1) -> TupleList2
> 
> #### Types
> 
> + Fun = fun((Term1 :: term()) -> Term2 :: term())
> + N = integer() >= 1 (1..tuple_size(Tuple))
> + TupleList1 = TupleList2 = [Tuple]
> + Tuple = tuple()
> 
> Returns a list of tuples where, for each tuple in TupleList1, the Nth element Term1 of the tuple has been replaced with the result of calling Fun(Term1).
> 
> [参照元](http://erlang.org/doc/man/lists.html#keymap-3)

#### Explain

タプルに対して指定した位置の要素に、引数で渡した関数を実行した結果が入れられた、新しいタプルのリストが返されます。


#### Example

```
1> IsProgrammer = fun(X) -> X =:= programmer end.
#Fun<erl_eval.6.17052888>
2> lists:keymap(IsProgrammer, 3, 
2> [
2>     {mike, 32, programmer}, {jim, 22, engineer}, {tim, 40, scientist},
2>     {bob, 36, officer}, {lod, 22, dentist}, {iwan, 27, evangelist},
2>     {simon, 36, typist}, {dag, 34, student}, {jack, 21, programmer},
2>     {john, 21, engineer}, {mike, 27, scientist}, {jim, 24, officer}
2> ]).
[{mike,32,true},
 {jim,22,false},
 {tim,40,false},
 {bob,36,false},
 {lod,22,false},
 {iwan,27,false},
 {simon,36,false},
 {dag,34,false},
 {jack,21,true},
 {john,21,false},
 {mike,27,false},
 {jim,24,false}]
```

この例ではタプルの3番目の要素がprogrammerならtrueに変換し、そうでなければfalseに変換した新しいタプルのリストを返します。



### keymember/3

#### Erlang公式ドキュメント

> #### keymember(Key, N, TupleList) -> boolean()
> 
> #### Types
> 
> + Key = term()
> + N = integer() >= 1 (1..tuple_size(Tuple))
> + TupleList = [Tuple]
> + Tuple = tuple()
> 
> Returns true if there is a tuple in TupleList whose Nth element compares equal to Key, otherwise false.
> 
> [参照元](http://erlang.org/doc/man/lists.html#keymember-3)

#### Explain

指定位置に`Key`を含むタプルがリスト中にあれば`true`、なければ`false`が返ってきます。


#### Example

```
1> lists:keymember(bob, 1, 
1> [{mike,32,programmer},
1>  {jim,22,engineer},
1>  {tim,40,scientist},
1>  {bob,36,officer},
1>  {lod,22,dentist},
1>  {iwan,27,evangelist},
1>  {simon,36,typist},
1>  {dag,34,student},
1>  {jack,21,programmer},
1>  {john,21,engineer},
1>  {mike,27,scientist},
1>  {jim,24,officer}   
1> ]).
true
2> lists:keymember(madscientist, 3,
2> [
2>  {mike,32,programmer},
2>  {jim,22,engineer},
2>  {tim,40,scientist},
2>  {bob,36,officer},
2>  {lod,22,dentist},
2>  {iwan,27,evangelist},
2>  {simon,36,typist},
2>  {dag,34,student},
2>  {jack,21,programmer},
2>  {john,21,engineer},
2>  {mike,27,scientist},
2>  {jim,24,officer}
2> ]).
false
```

最初の例では、1番目の要素が`bob`であるタプルがリスト中に含まれていますので、`true`が返されます。

次の例では、3番目の要素が`madscientist`のタプルはリスト中に含まれていないので、`false`が返されます。


### keymerge/3

#### Erlang公式ドキュメント

> #### keymerge(N, TupleList1, TupleList2) -> TupleList3
> 
> #### Types
> 
> + N = integer() >= 1 (1..tuple_size(Tuple))
> + TupleList1 = [T1]
> + TupleList2 = [T2]
> + TupleList3 = [(T1 | T2)]
> + T1 = T2 = Tuple
> + Tuple = tuple()
> 
> Returns the sorted list formed by merging TupleList1 and TupleList2. The merge is performed on the Nth element of each tuple. Both TupleList1 and TupleList2 must be key-sorted prior to evaluating this function. When two tuples compare equal, the tuple from TupleList1 is picked before the tuple from TupleList2.
> 
> [参照元](http://erlang.org/doc/man/lists.html#keymerge-3)

#### Explain

この関数は引数で与えられた二つのタプルリストをソートしてマージした状態で返します。ソートのキーは引数の`N`番目の要素になります。引数に与えられるタプルリストは事前にソートされていることが求められます。双方のタプルでキーの値が一致する場合、左側の引数のリストから取られたタプルが右のものに優先されます。


#### Example

```
1> lists:keymerge(1,  
1>   [{a, 1}, {c, 1}, {d, 1}],
1>   [{b, 2}, {d, 2}]).
[{a,1},{b,2},{c,1},{d,1},{d,2}]
2> lists:keymerge(1,
2>   [{x, 1}, {d, 1}, {e, 1}],
2>   [{t, 2}, {a, 2}, {d, 2}]).
[{t,2},{a,2},{d,2},{x,1},{d,1},{e,1}]
```

最初の例では、事前にソートされたタプルリストが引数として与えられ、マージされたリストが返されます。また、`[{d, 1}, {d, 2}]`のようにキーの値が一致するものは左側の引数に与えられたものが優先されています。

次の例では、事前にソートされていないタプルリストが引数として与えられていますが、返されるリストはマージされていない状態で返ってきます。


### keyreplace/4

#### Erlang公式ドキュメント

> #### keyreplace(Key, N, TupleList1, NewTuple) -> TupleList2
> 
> #### Types
> 
> + Key = term()
> + N = integer() >= 1 (1..tuple_size(Tuple))
> + TupleList1 = TupleList2 = [Tuple]
> + NewTuple = Tuple
> + Tuple = tuple()
> 
> Returns a copy of TupleList1 where the first occurrence of a T tuple whose Nth element compares equal to Key is replaced with NewTuple, if there is such a tuple T.
> 
> [参照元](http://erlang.org/doc/man/lists.html#keyreplace-4)

#### Explain

タプルリストの中で最も最初に現れた`N`番目の要素が`Key`であるタプルを引数で指定されたタプルに変更したリストを返します。


#### Example

```
1> lists:keyreplace(b, 1,
1> [                     
1>  {a, original},       
1>  {b, original},       
1>  {c, original},       
1>  {d, original},       
1>  {b, not_change}],
1> {b, changed}).        
[{a,original},
 {b,changed},
 {c,original},
 {d,original},
 {b,not_change}]
2> lists:keyreplace(x, 1,
2> [ 
2>  {a, original},
2>  {b, original},
2>  {c, original}],
2> {x, not_appear}).
[{a,original},{b,original},{c,original}]
```

最初の例では最も最初に現れる1番目の要素が`b`であるタプルが引数で与えられたタプルと交換されて返されます。最後に現れる同じ`Key`をもつタプルは変わっていません。

次の例では`Key`に一致するタプルが存在しないため、元のリストと同じ物が返ってきます。


### keysearch/3

#### Erlang公式ドキュメント

> #### keysearch(Key, N, TupleList) -> {value, Tuple} | false
> 
> #### Types
> 
> + Key = term()
> + N = integer() >= 1 (1..tuple_size(Tuple))
> + TupleList = [Tuple]
> + Tuple = tuple()
> 
> Searches the list of tuples TupleList for a tuple whose Nth element compares equal to Key. Returns {value, Tuple} if such a tuple is found, otherwise false.
> 
> > This function is retained for backward compatibility. The function lists:keyfind/3 (introduced in R13A) is in most cases more convenient.
> 
> [参照元](http://erlang.org/doc/man/lists.html#keysearch-3)

#### Explain

タプルリストの中で`N`番目の要素が`Key`である最初のタプルを探して、`{value, Tuple}`の形式で返します。見つからない場合は、`false`を返します。

なお、この関数は下位互換のために残っているものであり、R13Aより導入された`lists:keyfind/3`関数の方が便利です。


#### Example

```
1> lists:keysearch(c, 1, 
1> [
1>   {a, 1},
1>   {b, 1},
1>   {c, 1},
1>   {d, 1},
1>   {c, 2}]).
{value,{c,1}}
2> lists:keysearch(x, 1,
2> [                    
2>   {a, 1},            
2>   {b, 1},            
2>   {c, 1},            
2>   {d, 1},            
2>   {c, 2}]).          
false
```

最初の例では1番目の要素が`c`であるタプルを探して`{value, {c, 1}}`のように返します。二番目に現れるものは返しません。

次の例では条件に該当するタプルがないために`false`が返ってきます。


### keysort/2

#### Erlang公式ドキュメント

> #### keysort(N, TupleList1) -> TupleList2
> 
> #### Types
> 
> + N = integer() >= 1 (1..tuple_size(Tuple))
> + TupleList1 = TupleList2 = [Tuple]
> + Tuple = tuple()
> 
> Returns a list containing the sorted elements of the list TupleList1. Sorting is performed on the Nth element of the tuples. The sort is stable.
> 
> [参照元](http://erlang.org/doc/man/lists.html#keysort-2)

#### Explain

引数に渡されたタプルリストを`N`番目の要素でソートしたリストを返します。ソートは安定ソートです。


#### Example

```
1> lists:keysort(1,
1> [ 
1>   {x, 1},
1>   {r, 1},
1>   {t, 1},
1>   {x, 2},
1>   {c, 1},
1>   {s, 1},
1>   {r, 2},
1>   {a, 1},
1>   {x, 3}]).
[{a,1},{c,1},{r,1},{r,2},{s,1},{t,1},{x,1},{x,2},{x,3}]
```

ソートされた結果が返ってきます。安定ソートのため、重複するキーに対しては元の順番が維持されます。


### keystore/4

#### Erlang公式ドキュメント

> #### keystore(Key, N, TupleList1, NewTuple) -> TupleList2
> 
> #### Types
> 
> + Key = term()
> + N = integer() >= 1 (1..tuple_size(Tuple))
> + TupleList1 = [Tuple]
> + TupleList2 = [Tuple, ...]
> + NewTuple = Tuple
> + Tuple = tuple()
> 
> Returns a copy of TupleList1 where the first occurrence of a tuple T whose Nth element compares equal to Key is replaced with NewTuple, if there is such a tuple T. If there is no such tuple T a copy of TupleList1 where [NewTuple] has been appended to the end is returned.
> 
> [参照元](http://erlang.org/doc/man/lists.html#keystore-4)

#### Explain

タプルリストの中で一番最初に出てくる`N`番目の要素が`Key`であるタプルを、引数のタプルと交換したリストを返します。`Key`に一致するリストがない場合は、リストの最後に追加されて返されます。


#### Example

```
1> lists:keystore(b, 1,                                     
1>   [{a, origin}, {b, origin}, {c, origin}, {b, not_store}],
1>   {b, new_tuple}).
[{a,origin},{b,new_tuple},{c,origin},{b,not_store}]
2> lists:keystore(x, 1,                                      
2>   [{a, origin}, {b, origin}, {c, origin}, {b, not_store}],
2>   {x, not_found}).                                        
[{a,origin},
 {b,origin},
 {c,origin},
 {b,not_store},
 {x,not_found}]
```

最初の例では、1番目の要素が`b`であるタプルが、引数に与えられた新しいタプルに交換されたリストが返ってきます。

次の例では、該当するタプルが存在しないため、引数に与えられた新しいタプルがリストの最後に追加されて返ってきます。


### keytake/3

#### Erlang公式ドキュメント

> #### keytake(Key, N, TupleList1) -> {value, Tuple, TupleList2} | false
> 
> #### Types
> 
> + Key = term()
> + N = integer() >= 1 (1..tuple_size(Tuple))
> + TupleList1 = TupleList2 = [Tuple]
> + Tuple = tuple()
> 
> Searches the list of tuples TupleList1 for a tuple whose Nth element compares equal to Key. Returns {value, Tuple, TupleList2} if such a tuple is found, otherwise false. TupleList2 is a copy of TupleList1 where the first occurrence of Tuple has been removed.
> 
> [参照元](http://erlang.org/doc/man/lists.html#keytake-3)

#### Explain

タプルリストの中から初めて現れる`N`番目の要素が`Key`であるタプルを取り出し、`{value, Tuple, TupleList2}`の形式で返します。戻されるタプルリストには条件に一致したタプルは含まれていません。一致するものがない場合は、`false`が返されます。


#### Example

```
1> lists:keytake(b, 1, 
1> [{a, 1}, {b, 1}, {c, 1}, {b, 2}]).
{value,{b,1},[{a,1},{c,1},{b,2}]}
2> lists:keytake(x, 1,               
2> [{a, 1}, {b, 1}, {c, 1}, {b, 2}]).
false
```

最初の例では一番最初に現れる1番目の要素が`b`のタプルが取り出され、後から現れるものは残ったリストとともに返されています。

次の例では、条件に一致するタプルがないため、`false`が返されます。


### last/1

#### Erlang公式ドキュメント

> #### last(List) -> Last
> 
> #### Types
> 
> + List = [T, ...]
> + Last = T
> + T = term()
> 
> Returns the last element in List.
> 
> [参照元](http://erlang.org/doc/man/lists.html#last-1)

#### Explain

リストの最後の要素を返します。


#### Example

```
1> lists:last([1,2,3,4,5]).
5
2> lists:last([1]).
1
3> lists:last([]). 
** exception error: no function clause matching lists:last([]) (lists.erl, line 213)
```

最初の例ではリストの最後の要素`5`が返されます。

次の例では長さ1のリストを引数として渡して、最後の要素=たったひとつの要素である`1`が返されます。

最後の例では長さ0のリストを引数として渡します。この場合は例外が発生します。


### map/2

#### Erlang公式ドキュメント

> #### map(Fun, List1) -> List2
> 
> #### Types
> 
> + Fun = fun((A) -> B)
> + List1 = [A]
> + List2 = [B]
> + A = B = term()
> 
> Takes a function from As to Bs, and a list of As and produces a list of Bs by applying the function to every element in the list. This function is used to obtain the return values. The evaluation order is implementation dependent.
> 
> [参照元](http://erlang.org/doc/man/lists.html#map-2)

#### Explain

要素を返還する関数を引数に取ります。そしてリストの要素１つずつに関数を適用した結果が入ったリストが返されます。なお、実行順序は実装に依存しています。


#### Example

```
1> lists:map(fun(X) -> X rem 2 =:= 0 end, [1, 2, 3, 4, 5]).
[false,true,false,true,false]
```

引数に偶数かどうか判定する関数を渡します。引数のリストは1から5までの整数ですので、`false`と`true`が繰り返されるリストが返ってきます。


### mapfoldl/3

#### Erlang公式ドキュメント

> #### mapfoldl(Fun, Acc0, List1) -> {List2, Acc1}
> 
> #### Types
> 
> + Fun = fun((A, AccIn) -> {B, AccOut})
> + Acc0 = Acc1 = AccIn = AccOut = term()
> + List1 = [A]
> + List2 = [B]
> + A = B = term()
> 
> mapfoldl combines the operations of map/2 and foldl/3 into one pass. 
> 
> [参照元](http://erlang.org/doc/man/lists.html#mapfoldl-3)

#### Explain

`mapfoldl/3`関数は`map/2`関数と`foldl/3`関数を一つにまとめたような関数です。


#### Example

```
1> Multiply = fun ({R1, I1}, {R2, I2}) ->
1>     {R1 * R2 - I1 * I2, R1 * I2 + I1 * R2} end.
#Fun<erl_eval.12.17052888>
2> lists:mapfoldl(fun(X, A) -> Res = Multiply(X, A), {Res, Res} end,
2>   {1, 0},
2> [{1, -1}, {-1, 1}, {-1, -1}, {1, -1}]).
{[{1,-1},{0,2},{2,-2},{0,-4}],{0,-4}}
```

複素数`{R, I}`に対して、掛け算をする関数`Multiply`を定義し、`lists:mapfoldl/3`で初期値`{1, 0}`から左から順繰りに掛け算をしていきます。返されたタプルの左要素には掛け算の経過が、タプルの右側の要素には最終的な掛け算の結果が返っています。



### mapfoldr/3

#### Erlang公式ドキュメント

> #### mapfoldr(Fun, Acc0, List1) -> {List2, Acc1}
> 
> #### Types
> 
> + Fun = fun((A, AccIn) -> {B, AccOut})
> + Acc0 = Acc1 = AccIn = AccOut = term()
> + List1 = [A]
> + List2 = [B]
> + A = B = term()
> 
> mapfoldr combines the operations of map/2 and foldr/3 into one pass.
> 
> [参照元](http://erlang.org/doc/man/lists.html#mapfoldr-3)

#### Explain

`mapfoldr`関数は`map/2`関数と`foldr/3`関数を組み合わせた関数です。


#### Example

```
1> Multiply = fun ({R1, I1}, {R2, I2}) ->
1>     {R1 * R2 - I1 * I2, R1 * I2 + I1 * R2} end.
#Fun<erl_eval.12.17052888>
2> lists:mapfoldr(fun(X, A) -> Res = Multiply(X, A), {Res, Res} end,
2>   {1, 0},                                                        
2> [{1, -1}, {-1, 1}, {-1, -1}, {1, -1}]).                          
{[{0,-4},{2,-2},{-2,0},{1,-1}],{0,-4}}
```

先ほどと同じく複素数の計算をしています。返されたタプルの左側の順序が先ほどの`mapfoldl/3`関数と異なっていることがわかると思います。


### max/1

#### Erlang公式ドキュメント

> #### max(List) -> Max
> 
> #### Types
> 
> + List = [T, ...]
> + Max = T
> + T = term()
> 
> Returns the first element of List that compares greater than or equal to all other elements of List.
> 
> [参照元](http://erlang.org/doc/man/lists.html#max-1)

#### Explain

他のすべての要素より大きい最初の要素を返します。


#### Example

```
1> lists:max([0,-2,4,-6]).
4
```

リストの中の最大値である`4`が返されます。


### member/2

#### Erlang公式ドキュメント

> #### member(Elem, List) -> boolean()
> 
> #### Types
> 
> + Elem = T
> + List = [T]
> + T = term()
> 
> Returns true if Elem matches some element of List, otherwise false.
> 
> [参照元](http://erlang.org/doc/man/lists.html#member-2)

#### Explain

指定した要素と一致する要素があれば`true`を、なければ`false`を返します。


#### Example

```
1> lists:member({a,1}, [{b,1},{c,1},{a,2},{d,1},{a,1}]).
true
2> lists:member({a,1}, [{b,1},{c,1},{a,2},{d,1}]).      
false
```

最初の例では指定した要素`{a,1}`に一致する要素がリストにあるため`true`が返ってきます。

次の例では指定した要素`{a,1}`に一致する要素がリストにないため`false`が返ってきます。


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


