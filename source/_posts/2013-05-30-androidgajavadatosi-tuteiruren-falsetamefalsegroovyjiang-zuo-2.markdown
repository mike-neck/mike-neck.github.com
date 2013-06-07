---
layout: post
title: "AndroidがJavaだと思っている人のためのGroovy講座 - 2"
date: 2013-05-30 10:48
comments: true
categories: [groovy, java]
---

こんいちわ。みけです。

またも、タイトルが仰々しくてすみません。

本当に大したこと書かないです。

で、相変わらずAndroidまたもや出て来ません。

gradleも出て来ません。

でも、Android Studioの登場や、gradle-android-pluginで

gradleに興味を持たれた方には読んで貰いたいと思います。

以下、本題。

`Closure<V>`とはなんぞや
---

Closureについて僕がなんか言うと、

皆が混乱するので(それだけ僕もちゃんと説明できるほど理解していない…orz)

とりあえず[WikipediaのClosure](http://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AD%E3%83%BC%E3%82%B8%E3%83%A3)でも読んで下さい。

Javaでいえば、`Callable<V>`みたいなものです。

で、`Callable<V>`に比べて便利なのが

```java
    Callable<String> callable = new Callable<String>() {
        @Override
        public String call() {
            // some codes
            return result.toString();
        }
    };
```

みたいに書かなくてよいあたりです。

ちなみに`Callable<T>`と異なるところもあります。

それは、おいおい、説明します。


そろそろ`Closure<V>`のサンプル見せてくれよ
===

というわけで、適当にサンプルをみつくろってみました。

#### 単純な値を返すだけのClosure

```groovy
def closure = {
    'Hello, Closure!'
}

assert closure() == 'Hello, Closure!'
```

そうそう、**groovyでは`return`を省略することができます**。

その場合、最後に評価された式の値が`return`されます。

上記のClosureではClosureの最終式`'Hello, Closure!'`が評価され、

その値`'Hello, Closure!'`が返されます。


#### 引数をとるClosure

```groovy
def hello = {name ->
    'Hello, ' + name
}

assert 'Hello, mike' == hello('mike')
assert 'Hello, null' == hello()
```

`Closure<V>`と`Callable<V>`との決定的な違いが、
(大した違いではないが)

引数を与えることができる点です。

これを`Callable<V>`でやろうとすると次のようになります。

```java
public class HelloTest {

    public class Hello implements Callable<String> {

        private final String name;

        public Hello(String name) {
            this.name = name;
        }

        @Override
        public String call() throws Exception {
            return "Hello, " + name;
        }
    }

    @Test
    public void testHello () throws Exception {
        assertThat(new Hello("mike").call(), is("Hello, mike"));
        assertThat(new Hello(null).call(), is("Hello, null"));
    }
}
```

#### Closureを返すClosure

```groovy
def counter = {int offset ->
    return {
        offset++
    }
}

def closure = counter(0)

(0..100).each {
    assert it == closure()
}
```

初期値の0からカウントするカウンターのようなクロージャーが返ってきます。

型としては`Closure<Closure<Integer>>`といったところでしょうか…

で、これをみると一つ気持ち悪いところがありますね。

**元の値ってどうなってしまうの？**

```groovy
def closure = {String hello ->
    return {String name ->
        hello += name
    }    
}

def original = 'Hello, '
def message = closure(original)

assert message('mike') == 'Hello, mike'
assert message('!') == 'Hello, mike!'
assert original == 'Hello, '
```

というように、元の値は壊れませんが、

返されるClosureが元々持っていた値は壊れていきます。

ところで、イテレーターとして`Closure<V>`を使うときに出てくる

#### `it`ってなんやねん？

gradleのプラグイン宣言などで、ときどきこんな記述が出てきますね

```groovy
['java', 'groovy'].each {
    apply plugin : it
}
```

で、この`it`ですが、コレクションの一つ一つの要素です。

あ、そんなこと言われなくても知ってたって(´・ω・｀)

まあ、そんなことは言わずに、すこし試してみましょう。

```groovy
(1..4).each {
    println "it is ${it}."
}
```

実行結果はこんな感じです。

```
it is 1.
it is 2.
it is 3.
it is 4.
```

これって、でもどういうふうに呼ばれているの？

groovyの`DefaultGroovyMethods`クラスを介して呼ばれています。


##### 自分でも`it`でアクセスできるイテレーター作ってみたい

というわけで、`Closure<V>`を引数にとるメソッドを書いてみました。

`it`で要素にアクセスできて、ちょっと嬉しいですね。

```groovy
Integer.metaClass.define {
    collect = {Closure closure ->
        return new IntRange(0, delegate).collect {
            closure(it)
        }
    }
}

assert 2.collect {
    it * 2
} == [0, 2, 4]
```

というわけで、`it`は一つ一つの要素であることが事象として理解していただけたと思います。

で、`it`っていつ割り当てられるの？ということで、ソースをざっと見ていたんですが、

見つかりませんでした(´・ω・｀)


なお調べている過程で僕も初めて知ったのですが、

`Closure<V>`は抽象クラスで、

コンパイル時に実際のクラスが生成されているようです。

まだまだ僕も勉強が足りませんね…


で、このへんが気になりだすと、もっと気になるのが…

#### Closureのスコープってどうなのよ

```groovy
def outside = 'This is outside'
def doSomething = {println 'do something'}
def closure = {
    def inside = 'This is inside'
    doSomething()
    println outside
    println inside
}

closure()

doSomething()
println outside
println inside
```

これを実効すると次のような結果が得られます。

```
do something
This is outside
This is inside
do something
This is outside
Exception thrown
5 31, 2013 8:25:06 午後 org.codehaus.groovy.runtime.StackTraceUtils sanitize
WARNING: Sanitizing stacktrace:
groovy.lang.MissingPropertyException: No such property: inside for class: ConsoleScript83
	at org.codehaus.groovy.runtime.ScriptBytecodeAdapter.unwrap(ScriptBytecodeAdapter.java:50)
	at org.codehaus.groovy.runtime.callsite.PogoGetPropertySite.getProperty(PogoGetPropertySite.java:49)
	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.callGroovyObjectGetProperty(AbstractCallSite.java:231)
	at ConsoleScript83.run(ConsoleScript83:14)
```

`Closure<V>`の`closure`の中からは、

フィールド`outside`を参照することや、`doSomething`を実行することはできますが、

スクリプト本体からは`closure`内部の`inside`にアクセスすることはできません。

まあ、大体ご想像されていたとおりだと思います。


#### 実は`Callable<V>`だった`Closure<V>`

さて、ここまでは`Callable<V>`との対比で`Closure<V>`を見て来ましたが、

`Closure<V>`は実は`Callable<V>`を実装した抽象クラスです。

まず、`Closure<V>`のクラスファイルを見てみましょう。

```java Closure.java
public abstract class Closure<V> extends GroovyObjectSupport implements Cloneable, Runnable, GroovyCallable<V>, Serializable {
    // some codes
}
```

`Closure<V>`はインターフェース`GroovyCallable<V>`を実装しています。

で、この`GroovyCallable<V>`ってなんやねんというと…

```java GroovyCallable.java
public interface GroovyCallable<V> extends Callable<V> { }
```

ってことで、`Callable<V>`を継承しているインターフェースになります。

まとめると`Closure<V>`は`Callable<V>`の実装クラスということになります。

では試してみましょう。

```groovy
import java.util.concurrent.*

def service = Executors.newFixedThreadPool(3)

def callable = {int sec ->
    return {
        println "${new Date().format('yyyy/MM/dd hh:mm:ss')} : Sleeping ${sec} seconds..."
        Thread.sleep(sec * 1000)
        "${new Date().format('yyyy/MM/dd hh:mm:ss')} : This is ${sec}" as String
    }
}

assert service.invokeAll((10..1).collect {
    callable(it) as Callable<String>
}).each {
    println it.get()
}.size() == 10
```

この実行結果は次のようになります。

```
2013/05/31 02:13:56 : Sleeping 10 seconds...
2013/05/31 02:13:56 : Sleeping 8 seconds...
2013/05/31 02:13:56 : Sleeping 9 seconds...
2013/05/31 02:14:04 : Sleeping 7 seconds...
2013/05/31 02:14:05 : Sleeping 6 seconds...
2013/05/31 02:14:06 : Sleeping 5 seconds...
2013/05/31 02:14:11 : Sleeping 4 seconds...
2013/05/31 02:14:11 : Sleeping 2 seconds...
2013/05/31 02:14:11 : Sleeping 3 seconds...
2013/05/31 02:14:13 : Sleeping 1 seconds...
2013/05/31 02:14:06 : This is 10
2013/05/31 02:14:05 : This is 9
2013/05/31 02:14:04 : This is 8
2013/05/31 02:14:11 : This is 7
2013/05/31 02:14:11 : This is 6
2013/05/31 02:14:11 : This is 5
2013/05/31 02:14:15 : This is 4
2013/05/31 02:14:14 : This is 3
2013/05/31 02:14:13 : This is 2
2013/05/31 02:14:14 : This is 1
```

全然、問題なく`Callable<V>`として動いているのが確認できると思います。


で、別にここまではなんだjavaじゃん、というわけですが、

ここからがほぼ本題です。

#### delegate

delegateがgroovyの`Closure<V>`の柔軟性をもたらしていることに疑いはありません。

まずは使用例から…

##### `groovy.xml.MarkupBuilder`

```groovy
import groovy.xml.*

def w = new StringWriter()
def doc = new MarkupBuilder(w)

def border = 'border : solid 1px #ccc;'
def background = 'background-color : rgba(239, 239, 255, 0.7);'
def padding = 'padding : 5px;'
def radius = 'border-radius : 5px;'

doc.html(lang : 'ja') {
    head {
        title 'test page'
    }
    body {
        h3 'Hello Groovy!'
        p (style : "${border}${background}${padding}${radius}",
                'This document is made by groovy.')
    }
}

println w.toString()
```

これを実効すると、次のようなhtmlファイルができます。

```html
<html lang='ja'>
  <head>
    <title>test page</title>
  </head>
  <body>
    <h3>Hello Groovy!</h3>
    <p style='border : solid 1px #ccc;background-color : rgba(239, 239, 255, 0.7);padding : 5px;border-radius : 5px;'>This document is made by groovy.</p>
  </body>
</html>
```

`MarkupBuilder`というクラスは`groovy.util.BuilderSupport`クラスを継承していて、

存在しないようなメソッドを実行する際に、

```java
protected Object doInvokeMethod(String methodName, Object name, Object args)
```

を介して実行します。

そして、このメソッドが内部で`Node`を作っていきます。

そういうわけで、上記のコードの11行目の`html`メソッドが実行されて、

`"html"`というノードが作成されるのはわかるかと思います。

問題は`html`というメソッドが引数として受け入れた`Closure<V>`が

`head`というメソッドや`body`メソッドを実行しています。

さきほどの`Closure`のスコープに戻るとこの場合、

`head`や`body`というメソッドはないので、

ここで`MissingMethodException`が発生しそうですが、

発生しません。

何故でしょうか？


**これの謎を解く鍵がdelegateです。**


##### `Closure#setDelegate`と`Closure#setResolveStrategy`

`Closure<V>`には変数名やメソッド名をどのオブジェクトから参照するのかを決定することができます。

そのオブジェクトへの参照がdelegateになります。

では、delegateをわざとらしいほど強調したスクリプトを見てみます。

```groovy
setProperty 'x', 1000
setProperty 'y', 500
setProperty 'exec', {println "x : ${x}, y : ${y}"}

def map = [x : 10, y : 5, exec : {println "x = ${x}, y = ${y}"}]
def closure = {
    println "x -> ${x}"
    println "y -> ${y}"
    print 'delegate exec -> '
    exec.call()
    print 'owner exec -> '
    exec()
}
closure.delegate = map
closure.resolveStrategy = Closure.DELEGATE_FIRST
closure()
```

では、実行してみましょう。

```
x -> 10
y -> 5
delegate exec -> x = 1000, y = 500
owner exec -> x : 1000, y : 500
```

面白い結果が返ってきました。

`closure`内部で`x`と`y`の値は`map`から取得されたものになっています。

一方、`exec`に関しては二つの結果が出てきています。

これは、15行目と16行目で行われている`closure`のdelegateの設定によって説明出来ます。

`closure`がフィールド名を解決する仕組みは下記のようになっています。

+ `Closure.DELEGATE_FIRST`でdelegateされたオブジェクトから順番に解決していく
+ delegateされたオブジェクト`map`から変数を解決する

ここから、7行目、8行目の`x`と`y`は`map`由来のものであることがわかります。

また10行目と12行目の`exec`が異なる結果となっているのは、

10行目の`exec`はフィールド名としてまず解決された後に、

`Closure<V>`が実行されています。

一方、12行目の`exec`ではメソッドとして(`map`(クラスは`java.util.Map`)にはメソッド`exec`がない)

名前解決をします。そのため`closure`の`OWNER`であるスクリプトの方の`exec`が参照されます。

また、10行目と12行目の`exec`が参照する`x`と`y`が1行目と2行目で設定されている`x`と`y`なのは

次の理由からです。

+ `map.exec`のdelegateが設定されてない
+ `exec`のdelegateが設定されていない


##### `MarkupBuilder`再び

さきほどの`MarkupBuilder`に戻ります。

`MarkupBuilder#doInvokeMethod(String, Object, Object)`メソッドでは

どのような処理がされているのかというと…

```java MarkupBuilder.java
// this is partial codes
protected Object doInvokeMethod(String methodName, Object name, Object args) {
    Object node = null;
    Closure closure = null;
    
    // creating node and attach it to variable node.
    // getting closure from arg and attach it to variable closure.
    if (closure != null) {
        // stack operation
        setClosureDelegate(closure, node);
        closure.call();
        // stack operation
    }
    // node create completion and return it
}
protected void setClosureDelegate(Closure closure, Object node) {
    closure.setDelegate(this);
}
```

9行目で`setClosureDelegate`メソッドを呼び出し、

15行目からの`setClosureDelegate`では、

`closure`のdelegateに`this`(つまり`MarkupBuilder`のインスタンス)を割り当てています。

その結果10行目の`closure.call()`では、

メソッドおよびフィールドの名前解決が`MarkupBuilder`のインスタンスから行われるます。

その結果、`MarkupBuilder`の例で`head`とか`body`といったメソッドが、

`MarkupBuilder`のインスタンスに対して呼び出されるということになります。


さて、翻って

gradleのtaskメソッド…
===

でも、`Closure<V>`が使われていますね。

```groovy build.gradle
task myTask {
    description = 'this is myTask'.
    println 'this is not task, but configuration.'
    doLast {
    	println 'finished myTask'
    }
}
```

gradleのandroidサポートが本格的になってからgradleを始めた人の中には

上記のようなgradleの`Project`クラスのメソッド`task(Object, Closure)`の

`Closure`部分をタスクだと思っている人がかなりの割合でいると思いますが、

この`Closure`部分は**タスクではなく、設定です**。

こう書いた場合にタスクはgradleコマンドを書いた時に必ず実行されると

言っていると、元からgroovyをやっている人とは話がかみあわなくなるので、

気をつけて下さい。

では、ちょっとソースコードを覗いてみます。

```java AbstractProject.java
// this is partial codes
public Task task(String task, Closure configureClosure) {
    return taskContainer.create(task).configure(configureClosure);
}
```

`AbstractProject#task(String, Closure)`では、まず`Task`が作成された後に、

`Task#configure(Closure)`が呼び出されます。

```java AbstractTask.java
// this is partial codes
public Task configure(Closure closure) {
    return ConfigureUtil.configure(closure, this, false);
}
```

`Task#configure(Closure)`では`ConfigureUtil#configure(Closure, Task, boolean)`が呼び出されます。

```java ConfigureUtil.java
// this is partial codes
public static <T> T configure(Closure configureClosure,
        T delegate,
        boolean configureableAware) {
    return configure(configureClosure,
            delegate,
            Closure.DELEGATE_FIRST,
            configureableAware);
}
private static <T> T configure(Closure configureClosure,
        T delegate,
        int resolveStrategy,
        boolean configureableAware) {
    ClosureBackedAction<T> action = new ClosureBackedAction<T>(
            configureClosure,
            resolveStrategy,
            configureableAware); 
    action.execute(delegate);
    return delegate;
}
```

ここから、`org.gradle.api.internal.ClosureBackedAction<T>`により、

`Closure`が実行されます。

```java ClosureBackedAction.java
// this is partial codes
public ClosureBackedAction(Closure closure,
        int resolveStrategy,
        boolean configureableAware) {
    this.closure = closure;
    this.configureableAware = configureableAware;
    this.resolveStrategy = resolveStrategy;
}
public void execute(T delegate) {
    // check closure is not null
    // checking cinfgureableAware is false
    Closure copy = (Closure) closure.clone();
    copy.setResolveStrategy(resolveStrategy);
    copy.setDelegate(delegate);
    if (copy.getMaximumNumberOfParameters() == 0) {
        copy.call();
    } else {
        copy.call(delegate);
    }
}
```

12〜14行目でdelegateの設定、16行目or17行目で`Closure`の実行がなされていますね。

で、`Closure`のdelegateにはタスクが設定されています。

したがって、少し前で説明した通り、

`Closure`の実行において変数名の解決は`Task`のインスタンスから順番に解決されていきます。

で、これが最終的にタスクの設定になるわけです。

(結構たらい回しにされていてイライライする方がいらっしゃるかもしれませんが、まあ、テスタビリティを上げるためにこうなっています。)

というわけで、**まとめ**


`Closure<V>`の仕組みがわかると[Gradle DSL](http://www.gradle.org/docs/current/dsl/)の理解が容易になります。
---

冒頭でgradleの話しませんって言ったな、あれは、嘘だ。


{% render_partial _includes/post/post_footer.html %}

