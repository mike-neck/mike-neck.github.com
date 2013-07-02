---
layout: post
title: "JavaFXをヘッドレスブラウザーとして使うための基本テク #javafx"
date: 2013-07-02 10:28
comments: true
categories: java
---

こんにちわ、みけです。

JavaFXをヘッドレスブラウザーとして使おうとして、

いろいろとWebのリダイレクトにハマりまくっています。

で、今回はJavaFXをJavaFXのスレッド以外のスレッドから扱うTipsを集めました。


Goal
---

#### 期待していいこと

+ JavaFXの操作をJavaFXの外からする方法


#### 期待してはいけないこと

+ GUIプログラミングの云々かんぬん


Basics
---

JavaFXのアプリケーションのサンプル的な起動方法は次のとおりです。

```java SampleApplication.java
import javafx.application.Application;
import javafx.stage.Stage;
// この中のmainメソッドからアプリケーションを起動しちゃう
public class SampleApplication extends Application {
    public static void main(String... args) {
        Application.launch(SampleApplication.class);
    }
    @Override
    public void start(Stage stage) throws Exception {
        // do something.
    }
}
```

さて、この方法を採用している限りにおいては、

JavaFXのスレッドがどうのこうのでハマることはありません。

今回のテーマはJavaFXアプリケーションの外と内をわけることにあります。

JavaFXアプリケーションはバックグラウンドで実行します。

そのためには`java.util.concurrent.ExecutorService`を用います。

```java ApplicationLauncher.java
import javafx.application.Application;
import javafx.application.Platform;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
// 先ほどのSampleApplicationを起動します。
public class ApplicationLauncher {
    private static final ExecutorService SERVICE =
            Executors.newFixedThreadPool(1);
    public static void main(String... args) {
        // launch application
        SERVICE.submit(new Runnable(){
            @Override
            public void run() {
                Application.launch(SampleApplication.class);
            }
        });
        // do something
        // shutdown application
        Platform.exit();
        SERVICE.shutdown();
    }
}
```


`Platform#RunLater(java.lang.Runnable)`
---

JavaFX上でのみ動作するオブジェクトへの操作は`Platform#runLater(java.lang.Runnable)`を

通じて行います。

```java ControllerStimulation.java
import javafx.application.Platform;
public class ControllerStimulation {
    // JavaFXのcontrollerクラスのインスタンス
    private SampleController controller;
    // Platform#runLater(Runnable)を通じてcontrollerクラスへの操作を行う。
    public void simulateClickButton () {
        Platform.runLater(new SimulateClickButton());
    }
    // 実際にcontrollerクラスを操作するRunnable
    private class SimulateClickButton implements Runnable {
        @Override
        public void run () {
            controller.clickButton(null);
        }
    }
}
```


戻り値を利用したい場合
---

御存知の通り、`java.lang.Runnable#run()`は値を返しませんので、

値の受け渡しには`java.util.concurrent.BlockingQueue<T>`を使うことになります。

`Platform#runLater`が`java.util.concurrent.Callable<T>`を引数にとって、

`java.util.concurrent.Future<T>`を返してくれるといいんですけどね…

```java JavascriptExecution.java
import javafx.application.Platform;
import javafx.scene.web.WebEngine;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
public class JavascriptExecution {
    // controllerクラスのインスタンス
    private SampleController controller;
    public String callJavascript(String javascript) {
        final BlockingQueue<String> queue = new LinkedBlockingQueue<>();
        Platform.runLater(new CallJavascript(queue, javascript));
        try {
            String result = queue.take();
            return result;
        } catch(InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
    private class CallJavascript implements Runnable {
        private final BlockingQueue<String> queue;
        private final String script;
        CallJavascript(final BlockingQueue<String> queue, String script) {
            this.queue = queue;
            this.script = script;
        }
        @Override
        public void run() {
            WebEngine engine = controller.getEngine();
            String result = (String) engine.executeScript(script);
            try {
                queue.put(result);
            } catch(InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```

実際はJavascriptなどでエラーがあると、`netscape.javascript.JSException`がthrowされるので、

`Platform#runLater(java.lang.Runnable)`で`java.lang.RuntimeException`を検知して、

Applicatoinを終了させるような仕組みを組み込んでおくことが望まれるのですが、

それはここでの話を逸脱するし、誰得な気がするので、

やめておく…


以下、実際に僕が書いているコード

<a href="https://googledrive.com/host/0B4hhdHWLP7RRQVYweDh2QU1LVk0" target="_blank"><img src="//googledrive.com/host/0B4hhdHWLP7RRQVYweDh2QU1LVk0" style="width : 450px;"></a>



{% render_partial _includes/post/post_footer.html %}

