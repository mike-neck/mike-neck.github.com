---
layout: post
title: "gradleとJUnitのEnclosedの話"
date: 2013-05-25 23:19
comments: true
categories: java, gradle, groovy
---

みけです。

[先ほど書いたエントリー](http://mike-neck.github.io/blog/2013/05/25/javadehazimetesocketpuroguramuwoshu-itemita/)でテストをgradleで走らせたのですが、

<blockquote class="twitter-tweet" lang="ja"><p>gradleでテストした時に謎のclassMethodとかいうテストが勝手に挟み込まれて、落ちて困っている</p>&mdash; もちださんさん (@mike_neck) <a href="https://twitter.com/mike_neck/status/338275875294441472">2013年5月25日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

という状態が発生してました。

具体的には、

```java
package org.mikeneck.multithreads;

import org.junit.*;
import org.junit.experimental.runners.Enclosed;
import org.junit.rules.TestName;
import org.junit.runner.RunWith;

import java.io.IOException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.*;

import static org.hamcrest.Matchers.is;
import static org.junit.Assert.assertThat;

/**
 * @author mike
 */
@RunWith(Enclosed.class)
public class SimpleSocketTest {

    public static class SingleClient {

        private static final ExecutorService SERVICE = Executors.newFixedThreadPool(1);

        private static final int PORT = 12521;

        private static final String LOCALHOST = "localhost";

        private SimpleClient client;

        @Rule
        public TestName testName = new TestName();

        @BeforeClass
        public static void start () throws IOException {
            SERVICE.execute(new SimpleServer(PORT));
        }

        @Before
        public void setup () throws IOException {
            client = new SimpleClient(LOCALHOST, PORT);
        }

        @After
        public void tearDown () throws Exception {
            System.out.println(testName.getMethodName() + " is closing");
            client.close();
        }

        @AfterClass
        public static void end () throws IOException {
            new SimpleClient(LOCALHOST, PORT).open().bye();
        }

        @Test
        public void socketProcessing () throws IOException {
            client.open();
            String message = client.sendMessage("Hello");
            System.out.println("Message from Server [" + message + "]");
            assertThat(message, is("Hello"));
            System.out.println("Assertion ends.");
        }
    }
}
```

というテストに対して、

<img src="https://googledrive.com/host/0B4hhdHWLP7RRaS15VXNZRTNVWUU" style="width : 424px; height : 262px;" />

という感じで、謎の`classMethod`というテストが追加されていて、

実行されてテストが落ちてしまうようです。


先駆者はいた
---


とりあえず、`gradle`、`Enclosed`、`Junit`でググっていたところ、

次の二つのエントリーを発見しました。

+ [GradleでEnclosedのテストが二回実行されるんだ](http://d.hatena.ne.jp/irof/20120430/p1)
+ [GradleでEnclosedテストが2回実行されることの対策](http://d.hatena.ne.jp/shuji_w6e/20120808/1344386399)

というわけで、`gradle`と`Enclosed`の相性がわるいっぽい…

gradleでEnclosedなテストをする時の対策
---

というわけで、[@shuji_w6e](https://twitter.com/shuji_w6e)さんのページによると

**テストの実行時に除外クラスを指定すること。**

だそうです。


```groovy build.gradle
apply plugin : 'groovy'
apply plugin : 'idea'

group = 'org.mikeneck.multithreads'
version = '1.0'

def compatibility = 1.7

sourceCompatibility = compatibility
targetCompatibility = compatibility

repositories {
	mavenCentral ()
}

dependencies {
	compile 'org.codehaus.groovy:groovy-all:2.1.3'
	testCompile ('junit:junit:4.11') {
	    exclude module : 'hamcrest-core'
	    exclude module : 'hamcrest'
	}
	testCompile 'org.hamcrest:hamcrest-library:1.+'
}

test {
    exclude '**/*$*'
}
```

とりあえず、テストクラスを除外してみました。

結果
---

<img src="https://googledrive.com/host/0B4hhdHWLP7RRczBZcy04U2tNMms" style="width : 424px; height : 252px;"" />


という感じで、テストが通りました。


{% render_partial _includes/post/post_footer.html %}

