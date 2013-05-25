---
layout: post
title: "JavaではじめてSocketプログラムを書いてみた"
date: 2013-05-25 21:13
comments: true
categories: java
---

みけです。

jjugのメーリングリストにconcurrentに関する質問が着ていたので、

答えようと思ってみたんですが、

Socketを使ったプログラムで、

**そういえば僕、Socketを使ったプログラム書いたこと無いな**

と思い至り、書いてみることにしました。


SocketServerのほう
---

`java.net.ServerSocket`を使って書くようです。


```java SimpleServer.java
package org.mikeneck.socket;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @author mike_neck
 */
public class SimpleServer implements Runnable {

    private final ServerSocket serverSocket;

    public SimpleServer(int port) throws IOException {
        serverSocket = new ServerSocket(port);
    }

    @Override
    public void run() {
        while(true) {
            try (Socket socket = serverSocket.accept();
                 BufferedReader reader = new BufferedReader(
                     new InputStreamReader(socket.getInputStream()));
                 PrintStream writer = new PrintStream(socket.getOutputStream())
            ) {
                String line = reader.readLine();
                writer.println(line);
                if (line.equals("BYE")) {
                    break;
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```


SocketClientのほう
---

クライアントアプリケーションの方は`java.net.Socket`を用いるようです。

```java SimpleClient.java
package org.mikeneck.multithreads;

import java.io.BufferedReader;
import java.io.DataOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.Socket;

/**
 * @author mike_neck
 */
public class SimpleClient implements AutoCloseable {

    private final Socket socket;

    private DataOutputStream output;

    private BufferedReader input;

    public SimpleClient(String hostname, int port) throws IOException {
        socket = new Socket(hostname, port);
    }

    public SimpleClient open () throws IOException {
        output = new DataOutputStream(socket.getOutputStream());
        input = new BufferedReader(
            new InputStreamReader(socket.getInputStream()));
        return this;
    }

    public String sendMessage(String message) throws IOException {
        output.writeBytes(message + '\n');
        String line;
        if ((line = input.readLine()) != null) {
            return line;
        } else {
            return "";
        }
    }

    public void bye () throws IOException {
        output.writeBytes("BYE\n");
    }

    @Override
    public void close() throws Exception {
        if (!socket.isClosed()) {
            socket.close();
        }
    }
}

```

SocketServerでの疑問
---

と、まあ簡単なプログラムを書いたのですが、

二回メッセージを送る場合はどうなるのかとか、疑問が残りますね。

で、テストを書くわけですが…

案の定落ちました。

```java SendManyMessages.java
// import とか packageとか省略
public static class SendManyMessages {

    private static final ExecutorService SERVICE = Executors.newFixedThreadPool(1);

    private static final int PORT = 14541;

    private static final String LOCALHOST = "localhost";

    @Rule
    public TestName testName = new TestName();

    @BeforeClass
    public static void start () throws IOException {
        SERVICE.execute(new SimpleServer(PORT));
    }

    @AfterClass
    public static void end () throws IOException {
        new SimpleClient(LOCALHOST, PORT).open().bye();
    }

    @ThisTestWillFail
    @Test
    public void send2times () throws IOException {
        SimpleClient client = new SimpleClient(LOCALHOST, PORT).open();

        assertThat(client.sendMessage("hello"), is("hello"));
        assertThat(client.sendMessage("good-bye"), is("good-bye"));
    }
}
```

```
java.net.SocketException: Broken pipe
	at java.net.SocketOutputStream.socketWrite0(Native Method)
	at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:109)
	at java.net.SocketOutputStream.write(SocketOutputStream.java:132)
	at java.io.DataOutputStream.writeBytes(DataOutputStream.java:276)
	at org.mikeneck.multithreads.SimpleClient.sendMessage(SimpleClient.java:31)
	at org.mikeneck.multithreads.SendManyMessages.send2times(SendManyMessages.java:37)
```

うん、まあ、今回は別になんか状態を持つようなサーバーを作りたいわけではないし、

そういうサーバーを作りたい場合は、まあ、そういう工夫をすればいいということだけ覚えておこう。

なお、他に書いたテストもありますが、

面倒なので、gistにあげておきました。

[こちらをどうぞ](https://gist.github.com/mike-neck/5649095)

{% render_partial _includes/post/post_footer.html %}

