---
layout: post
title: "GradleでHerokuのアプリを作ってみる"
date: 2014-06-11 17:51
comments: true
categories: gradle, java
image: http://googledrive.com/host/0B4hhdHWLP7RRb2JjNjFTYUhILTQ
---

<img src="//googledrive.com/host/0B4hhdHWLP7RRb2JjNjFTYUhILTQ" style="width : 400px;"/>

久々にブログ書いています。

こんにちわ、みけです。

昨年の7月〜8月頃から、家をでるのが困難になり、

かつ何の活動をするのも億劫になっていたため、

ブログすらまともに書いていませんでした。


GradleでHerokuにアプリケーションをデプロイする
---

**Gradle Heroku**で検索をするとGradleで作ったアプリケーションを

Herokuにデプロイする方法が出てきます。

ただ、まあ、なんというか、みんなJetty embbededを使って```static void main```から

始まるアプリケーションが多かったので、

jetty-runnerでwarファイルを走らせる方法をここではまとめてあります。

…

もう既にJava好きな皆さんにとっては、なんというかもう常識的なことなので、

ここに書いてあることは当たり前なのかもしれませんが…

以下、必要なファイル毎に書いていきます。

* build.gradle
* Procfile
* system.properties


build.gradle
---

Herokuでjetty-runnerでwebアプリを動かす場合に注意する点は次のとおりです

### ```gradle wrapper```を実行しておく

実行する```gradle```のバージョンを一致させるために

```gradle wrapper```を実行しておきます。

```
$ gradle wrapper
```

これで、プロジェクトのディレクトリーに```gradlew```が作成されていることを確認しておきます。


### ```jetty```プラグインを使う

これは別に```war```プラグインでも構いませんが、

ローカルで動かしてテストしたいだろうと思いますので、

```jetty```プラグインを使います。

```groovy build.gradle
apply plugin : 'jetty'
```


### ```providedCompile``` configurationで```jetty-runner```を指定する

```jetty-runner```でHerokuアプリを走らせますので、

```providedCompile``` configurationで```jetty-runner```を指定しておきます。

```groovy build.gradle
repositories {
    mavenCentral()
}
dependencies {
    compile 'javax.servlet:javax.servlet-api:3.1.0'
    providedCompile ('org.eclipse.jetty:jetty-runner:9.2.0.v20140526') {
        exclude module : 'javax.servlet-api'
    }
    testCompile 'junit:junit:4.11'
}
```

---

また、```jetty-runner```をディレクトリー指定してダウンロードさせるために、

```Copy```タイプのタスクを作成します。


```groovy build.gradle
task copyJetty (type : Copy) {
    into "$buildDir/jetty"
    from configurations.providedCompile
}
```


### ```stage```タスクを用意する

HerokuでのGradleアプリケーションは```stage```タスクにてアプリケーションのビルドをします。

そのため、```stage```タスクに実行させておきたいタスクを記述しておきます。

```groovy build.gradle
task stage (dependsOn : ['clean', 'copyJetty'])
stage.finalizedBy build
```

上記で作成した```copyJetty```タスクは何故かファイルを上書きしてしまったので、

task finalizationで最終的に```build```タスクを実行させます。

これらをまとめた```build.gradle```は次のとおり

```groovy build.gradle
apply plugin : 'jetty'
repositories {
    mavenCentral()
}
dependencies {
    compile 'javax.servlet:javax.servlet-api:3.1.0'
    providedCompile ('org.eclipse.jetty:jetty-runner:9.2.0.v20140526') {
        exclude module : 'javax.servlet-api'
    }
    testCompile 'junit:junit:4.11'
}
task copyJetty (type : Copy) {
    into "$buildDir/jetty"
    from configurations.providedCompile
}
task stage (dependsOn : ['clean', 'copyJetty'])
stage.finalizedBy build
```


Procfile
---

```Procfile```にはアプリケーション実行のコマンドを記述しておきます。(あまりよくわかってない…)

最初、僕はこのファイルでgradleの```jettyRunWar```を実行させようとしていたのですが、

見事にコケてました。

```bash Procfile
web: java $JAVA_OPTS -jar build/jetty/jetty-runner-9.2.0.v20140526.jar --port $PORT build/libs/*.war
```

またHerokuのgradlewで作成されるwarファイルも、

なんかランダムな文字列になっているので、

warファイル指定にワイルドカードを使っています。


system.properties
---

ここにはJavaのバージョンを指定します。

僕はJava8を使いたかったのでjava8を指定します。

```java system.properties
java.runtime.version=1.7
```


Herokuへのアプリケーションのデプロイ
---

Herokuへのデプロイは皆さんが御存知の通りで、

僕が説明するまでのことはないでしょう。

- toolbeltをインストールする
- Herokuアカウントを作成する
- Herokuにssh-keyを登録する
- heroku側にアプリケーションを作成する
- herokuにpushする


heroku側にアプリケーションを作成するために、次のコマンドを打ちます

```bash command-line
$ heroku create
```

herokuにpushするために次のコマンドを打ちます

```bash command-line
$ git push heroku master
```

これでアプリケーションのデプロイが完了します。


{% render_partial _includes/post/post_footer.html %}

