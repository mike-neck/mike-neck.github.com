---
layout: post
title: "gradleのmaven-publishプラグインでライブラリーを発行する方法 - 1"
date: 2013-06-19 16:21
comments: true
categories: [groovy, java, gradle]
---

<img src="http://www.gradle.org/forum-assets/images/gradle_logo.gif"/>


こんにちわ、みけです。

ここ数日、gradleのmaven-publishプラグインにはまっていたので、

そのメモです。

maven-publishプラグインについて
===

以下のとおりにメモしていきます。

+ maven-publishプラグインの基礎
+ javadoc、sourcesを発行する
+ 複数回、成果物を発行する
+ pomを変更する
+ PGP署名ファイルの発行
+ 課題

maven-publishプラグインの基礎
---

maven-publishプラグインは任意のファイルを

任意のmavenレポジトリーにアップロードすることができるプラグインです。

次のようなビルドスクリプトでは、以下のようなartifactを発行することができます。

+ sample-project-1.0.jar
+ sample-project-1.0.pom
+ sample-project-1.0-doc.html

```groovy build.gradle
apply plugin : 'java'
apply plugin : 'maven-publish'
// project information
group = 'org.mikeneck.sample'
version = '1.0'
// publishing description
publishing {
    publications {
        sample(MavenPublication) {
            from components.java
            artifact ('document.html') {
                classifier = 'doc'
                extension  = 'html'
            }
        }
    }
    repositories {
        maven {
            url 'file://Users/mike/maven-sample-repo'
        }
    }
}
```

また、これらに付随して、それぞれのファイルのmd5ファイルとsha1ファイルも作成されます。

実際に実行してみます。

```
$ gradle clean test publish
:clean UP-TO-DATE
:compileJava
:processResources UP-TO-DATE
:classes
:compileTestJava
:processTestResources UP-TO-DATE
:testClasses
:test
:generatePomFileForSamplePublication
:jar
:publishSamplePublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.jar to repository remote at file:/Users/mike/maven-sample-repo
Transferring 1K from remote
Uploaded 1K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-doc.html to repository remote at file:/Users/mike/maven-sample-repo
Transferring 0K from remote
Uploaded 0K
:publish

BUILD SUCCESSFUL

Total time: 18.255 secs
```

pomファイルが発行されたのかどうかよくわかりませんが、

実際に発行されたディレクトリーを見てみます。

```
$ cd /Users/mike/maven-sample-repo/org/mikeneck/sample
$ ls
sample-project
$ cd sample-project/
$ ls
1.0    maven-metadata.xml    maven-metadata.xml.md5    maven-metadata.xml.sha1
$ cd 1.0/
$ ls
sample-project-1.0-doc.html	     sample-project-1.0.jar      sample-project-1.0.pom
sample-project-1.0-doc.html.md5  sample-project-1.0.jar.md5  sample-project-1.0.pom.md5
sample-project-1.0-doc.html.sha1 sample-project-1.0.jar.sha1 sample-project-1.0.pom.sha1
```

ということでコマンドを実行したあとの標準出力にはpomについての記述はありませんが、

ちゃんと発行されています。

### 規約

maven-publishプラグインでは以下のようなルールがあります。

+ 基本的なartifact名はproject名(ディレクトリの名前) + version番号
+ `classifier`に指定された文字列は上記のファイル名の最後に付与される
+ `extension`で指定された文字列は拡張子として付与される
+ `classifier` + `extension`での一意性のチェックが行われる


javadoc、sourcesを発行する
---

では、先ほどのサンプルに、ソースとjavadocを付与してみます。

```groovy build.gradle
apply plugin : 'java'
apply plugin : 'maven-publish'
// project information
group = 'org.mikeneck.sample'
version = '1.0'
// zip sources
task sourceJar(type : Jar) {
    from sourceSets.main.allJava
}
// zip javadocs
task javadocJar(type : Jar, dependsOn : javadoc) {
    from javadoc.destinationDir
}
// publishing description
publishing {
    publications {
        sample(MavenPublication) {
            from components.java
            artifact ('document.html') {
                classifier = 'doc'
                extension  = 'html'
            }
            artifact sourceJar {
                classifier = 'sources'
                extension  = 'jar'
            }
            artifact javadocJar {
                classifier = 'javadoc'
                extension  = 'jar'
            }
        }
    }
    repositories {
        maven {
            url 'file://Users/mike/maven-sample-repo'
        }
    }
}
```

DSLによれば、メソッド`artifact`にはtaskを引数にとることができ、指定したtaskの成果物を発行することができます。

上記の例では、`sourceJar`タスクによってjarファイルに固められたソースと、

`javadocJar`タスクに寄ってjarファイルに固められたjavadocが、

発行されるようになります。

では、実行してみましょう。

```
$ gradle clean test publish
:clean
:compileJava
:processResources UP-TO-DATE
:classes
:compileTestJava
:processTestResources UP-TO-DATE
:testClasses
:test
:generatePomFileForSamplePublication
:jar
:javadoc
:javadocJar
:sourceJar
:publishSamplePublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.jar to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 1K from remote
Uploaded 1K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-doc.html to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 0K from remote
Uploaded 0K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-sources.jar to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 1K from remote
Uploaded 1K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-javadoc.jar to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 33K from remote
Uploaded 33K
:publish

BUILD SUCCESSFUL

Total time: 25.84 secs
```

上記の標準出力からソースとjavadocが出力されていることがわかります。

実際に出力されたファイルを確認してみます。

```
$ cd /Users/mike/maven-sample-repo/org/mikeneck/sample/sample-project/1.0
$ ls -la
total 184
drwxr-xr-x  17 mike  mike   578B  6 19 17:40 .
drwxr-xr-x   6 mike  mike   204B  6 19 17:05 ..
-rw-r--r--   1 mike  mike   346B  6 19 17:40 sample-project-1.0-doc.html
-rw-r--r--   1 mike  mike    32B  6 19 17:40 sample-project-1.0-doc.html.md5
-rw-r--r--   1 mike  mike    40B  6 19 17:40 sample-project-1.0-doc.html.sha1
-rw-r--r--   1 mike  mike    33K  6 19 17:40 sample-project-1.0-javadoc.jar
-rw-r--r--   1 mike  mike    32B  6 19 17:40 sample-project-1.0-javadoc.jar.md5
-rw-r--r--   1 mike  mike    40B  6 19 17:40 sample-project-1.0-javadoc.jar.sha1
-rw-r--r--   1 mike  mike   927B  6 19 17:40 sample-project-1.0-sources.jar
-rw-r--r--   1 mike  mike    32B  6 19 17:40 sample-project-1.0-sources.jar.md5
-rw-r--r--   1 mike  mike    40B  6 19 17:40 sample-project-1.0-sources.jar.sha1
-rw-r--r--   1 mike  mike   1.1K  6 19 17:40 sample-project-1.0.jar
-rw-r--r--   1 mike  mike    32B  6 19 17:40 sample-project-1.0.jar.md5
-rw-r--r--   1 mike  mike    40B  6 19 17:40 sample-project-1.0.jar.sha1
-rw-r--r--   1 mike  mike   404B  6 19 17:40 sample-project-1.0.pom
-rw-r--r--   1 mike  mike    32B  6 19 17:40 sample-project-1.0.pom.md5
-rw-r--r--   1 mike  mike    40B  6 19 17:40 sample-project-1.0.pom.sha1
```

複数回、成果物を発行する
---

これまでの例では`sample`という発行タスクにいろいろなものを詰め込んでいました。

たとえば、javadocだけとか、sourcesファイルだけとか発行したい場合、

成果物の発行タスクを切り分けたいような場面があるかと思います。

その場合、`publications`の下の記述を変えることで、成果物の発行タスクを分けることができます。

```groovy build.gradle
apply plugin : 'java'
apply plugin : 'maven-publish'
// project information
group = 'org.mikeneck.sample'
version = '1.0'
// zip sources
task sourceJar(type : Jar) {
    from sourceSets.main.allJava
}
// zip javadocs
task javadocJar(type : Jar, dependsOn : javadoc) {
    from javadoc.destinationDir
}
// publishing description
publishing {
    publications {
        // only java archives
        sample(MavenPublication) {
            from components.java
        }
        // publish documents
        documents(MavenPublication) {
            artifact ('document.html') {
                classifier = 'doc'
                extension  = 'html'
            }
            artifact sourceJar {
                classifier = 'sources'
                extension  = 'jar'
            }
            artifact javadocJar {
                classifier = 'javadoc'
                extension  = 'jar'
            }
        }
    }
    repositories {
        maven {
            url 'file://Users/mike/maven-sample-repo'
        }
    }
}
```

これで、メインのjarを発行するタスクと、

ドキュメント類を発行するタスクを切り分けることができました。

実際、タスクにはどのようなものがあるか確認します。

```
$ gradle tasks
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Assembles and tests this project.

…中略…

Publishing tasks
----------------
publish - Publishes all publications produced by this project.
publishDocumentsPublicationToMavenLocal - Publishes Maven publication 'documents' to the local Maven repository.
publishDocumentsPublicationToMavenRepository - Publishes Maven publication 'documents' to Maven repository 'maven'.
publishSamplePublicationToMavenLocal - Publishes Maven publication 'sample' to the local Maven repository.
publishSamplePublicationToMavenRepository - Publishes Maven publication 'sample' to Maven repository 'maven'.
publishToMavenLocal - Publishes all Maven publications produced by this project to the local Maven cache.

…中略…

BUILD SUCCESSFUL

Total time: 21.853 secs
```

Publishing tasksには次のようなエントリーが入っています。

+ `publish` すべてを指定したレポジトリーに発行する
+ `publicDocumentsPublication…` `documents`で指定したアーカイブを発行します。
+ `publishSamplePublication…` `sample`で指定したアーカイブを発行します。
+ `publishToMavenLocal` すべてをmaven localレポジトリーに発行します。

という形で、`publishing/publications`で複数のアーカイブ発行を指定することで、タスクが生成されます。

さて、ここでは、`publish`タスクを実行してみたいと思います。

```
$ gradle clean test publish
:clean
:compileJava
:processResources UP-TO-DATE
:classes
:compileTestJava
:processTestResources UP-TO-DATE
:testClasses
:test
:generatePomFileForDocumentsPublication
:javadoc
:javadocJar
:sourceJar
:publishDocumentsPublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.pom to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 0K from remote
Uploaded 0K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-doc.html to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 0K from remote
Uploaded 0K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-sources.jar to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 1K from remote
Uploaded 1K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-javadoc.jar to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 33K from remote
Uploaded 33K
:generatePomFileForSamplePublication
:jar
:publishSamplePublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.jar to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 1K from remote
Uploaded 1K
:publish

BUILD SUCCESSFUL

Total time: 12.33 secs
```

標準出力を見るとわかりますが、**アルファベット順**にタスクが実行されます。

そして、もう一つ気になるところがありますね。

各タスクの前に、`generatePomFileFor[タスク名]`というタスクが実行されています。

これらが発行するpomは何かを調べてみます。

まず、すこしスクリプトを変更します。

```groovy build.gradle
apply plugin : 'java'
apply plugin : 'maven-publish'
// project information
group = 'org.mikeneck.sample'
version = '1.0'
// repositories
repositories {
    mavenCentral ()
}
// dependencies
dependencies {
    compile 'org.jsoup:jsoup:1.6.3'
    testCompile 'junit:junit:4.11'
}
// zip sources
task sourceJar(type : Jar) {
    from sourceSets.main.allJava
}
// zip javadocs
task javadocJar(type : Jar, dependsOn : javadoc) {
    from javadoc.destinationDir
}
// publishing description
publishing {
    publications {
        sample(MavenPublication) {
            from components.java
        }
        documents(MavenPublication) {
            artifact ('document.html') {
                classifier = 'doc'
                extension  = 'html'
            }
            artifact sourceJar {
                classifier = 'sources'
                extension  = 'jar'
            }
            artifact javadocJar {
                classifier = 'javadoc'
                extension  = 'jar'
            }
        }
    }
    repositories {
        maven {
            url 'file://Users/mike/maven-sample-repo'
        }
    }
}
```

`dependencies`を追加しました。

この状態で、sampleの方を実行してみます。

> なお、このタスクの実行において、タスクの指定に省略名を使用しています。
> 
> gradleでは`publishToMavenRepository`のような単語の頭文字が大文字になっているタスクを
> 
> 頭文字だけを選択して`pTMR`のように省略することができます。
> 


```
$ gradle clean pSPTMR
:clean
:generatePomFileForSamplePublication
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:publishSamplePublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.jar to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 1K from remote
Uploaded 1K

BUILD SUCCESSFUL

Total time: 2.498 secs
```

さて、この結果出力されたpomは次のようになります。

```xml sample-project-1.0.pom
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.mikeneck.sample</groupId>
  <artifactId>sample-project</artifactId>
  <version>1.0</version>
  <dependencies>
    <dependency>
      <groupId>org.jsoup</groupId>
      <artifactId>jsoup</artifactId>
      <version>1.6.3</version>
      <scope>runtime</scope>
    </dependency>
  </dependencies>
</project>
```

一方、documentの方を実行してみます。

```
$ gradle clean pDPTMR
:clean
:generatePomFileForDocumentsPublication
:compileJava
:processResources UP-TO-DATE
:classes
:javadoc
:javadocJar
:sourceJar
:publishDocumentsPublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.pom to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 0K from remote
Uploaded 0K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-doc.html to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 0K from remote
Uploaded 0K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-sources.jar to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 1K from remote
Uploaded 1K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-javadoc.jar to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 33K from remote
Uploaded 33K

BUILD SUCCESSFUL

Total time: 4.961 secs
```

この結果出力されたpomは次のとおり、`<dependencies>`〜`</dependencies>`の部分の記述がなくなります。

```xml sample-project-1.0.pom
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.mikeneck.sample</groupId>
  <artifactId>sample-project</artifactId>
  <version>1.0</version>
  <packaging>pom</packaging>
</project>
```

この違いは、前者のタスクにおいてdependenciesの情報を利用するのに対して、

後者はdependenciesを使わないことにあると勝手に理解しています。

したがって、複数回にわたって成果物を発行する場合は、

pomの生成について気をつけなければなりません。


pomを変更する
---

前回のポストで記述した通り、maven centralに登録するライブラリーについては、

発行するpomにいくつか追加情報を与えなければならない場合があります。

また、Jettyのservletを用いる場合は、

jetty-orbitという存在しないartifactを避けるために、

直接dependencyを書けない場合などがあります。

そのような場合に、pomを書き換える必要が生じます。


### publicationコンテナのpomオブジェクトを用いる

maven-publishプラグインではpublicationコンテナにて

pomオブジェクトを介して発行されるpomにアクセスすることができます。

先ほどのビルドスクリプトにpomを生成するタスクを追加します。

```groovy build.gradle
apply plugin : 'java'
apply plugin : 'maven-publish'
// project information
group = 'org.mikeneck.sample'
version = '1.0'
// repositories
repositories {
    mavenCentral ()
}
// dependencies
dependencies {
    compile 'org.jsoup:jsoup:1.6.3'
    testCompile 'junit:junit:4.11'
}
// zip sources
task sourceJar(type : Jar) {
    from sourceSets.main.allJava
}
// zip javadocs
task javadocJar(type : Jar, dependsOn : javadoc) {
    from javadoc.destinationDir
}
// publishing description
publishing {
    publications {
        sample(MavenPublication) {
            from components.java
        }
        documents(MavenPublication) {
            artifact ('document.html') {
                classifier = 'doc'
                extension  = 'html'
            }
            artifact sourceJar {
                classifier = 'sources'
                extension  = 'jar'
            }
            artifact javadocJar {
                classifier = 'javadoc'
                extension  = 'jar'
            }
        }
        // editing pom file with builder style
        pomOnly(MavenPublication) {
            pom.withXml {
                def node = asNode()
                node.children().last() + {
                    resolveStrategy = Closure.DELEGATE_FIRST
                    dependencies {
                        project.configurations.compile.dependencies.each {dep ->
                            dependency {
                                groupId dep.group
                                artifactId dep.name
                                version dep.version
                            }
                        }
                    }
                    licenses {
                        license {
                            name 'The Apache Software License, Version 2.0'
                            url 'http://www.apache.org/license/LICENSE-2.0.txt'
                            distribution 'repo'
                        }
                    }
                }
            }
        }
    }
    repositories {
        maven {
            url 'file://Users/mike/maven-sample-repo'
        }
    }
}
```

pomオブジェクトの`withXml`メソッドの引数の`Closure`は、

`org.gradle.api.XmlProvider`のメソッドを呼び出すことができます。

そして、`asNode()`メソッドによりpomファイルを`groovy.util.Node`の形で取得出来ます。

`asNode()`で返ってくる`Node`の一番トップの部分は`<project>`要素です。

この要素の子要素を取得し、最後の要素に`plus`メソッドで要素を追加します。

追加する`Closure`は、`groovy.util.NodeBuilder`と同等のDSLによって、

pomに要素を追加していくことができます。

では、このタスクを実行してみます。

```
$ gradle clean pPOPTMR
:clean
:generatePomFileForPomOnlyPublication
:publishPomOnlyPublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.pom to repository remote at file:/Users/mike/maven-sample-repo/
Transferring 1K from remote
Uploaded 1K

BUILD SUCCESSFUL

Total time: 5.813 secs
```

発行されたpomファイルは次のようになります。

```xml sample-project-1.0.pom
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.mikeneck.sample</groupId>
  <artifactId>sample-project</artifactId>
  <version>1.0</version>
  <packaging>pom</packaging>
  <dependencies>
    <dependency>
      <groupId>org.jsoup</groupId>
      <artifactId>jsoup</artifactId>
      <version>1.6.3</version>
    </dependency>
  </dependencies>
  <licenses>
    <license>
      <name>The Apache Software License, Version 2.0</name>
      <url>http://www.apache.org/license/LICENSE-2.0.txt</url>
      <distribution>repo</distribution>
    </license>
  </licenses>
</project>
```

PGP署名ファイルの発行 -- 書きかけ…
---

少し話題が飛びますが、

maven central repositoryにライブラリーを発行する場合、

各アーカイブファイルとpomファイルに対してPGP(Pretty Good Privacy)署名が必要となります。

### PGP署名って…？

PGP署名を簡単に説明すると以下のようになります。

+ 配布物を元に、配布元で非公開鍵で暗号化して署名を作る
+ 受け取り側で署名に対して公開鍵で復号化したものと、配布物とを比較する
+ 一致していれば配布物が正しいもの(改ざんされていない)と判定される

というファイルの信頼性を確認する仕組みです。

なお、PGPツールとしては、PGPの仕様RFC4880に準拠した、

[GnuPG(Gnu Privacy Guard)](http://www.gnupg.org)を使うのが一般的なようです。

なお、GnuPGの現在のバージョンは2.0です。

また、Javaでの実装ではBCPGが有名です。

また、gradle本題も`bcpg-jdk15-1.46`を利用しています。

[<img src="//googledrive.com/host/0B4hhdHWLP7RRNzktQnBvNjBnZE0" style="width : 300px;"/>](https://googledrive.com/host/0B4hhdHWLP7RRNzktQnBvNjBnZE0)

### 署名タスクを作成する

mavenプラグイン + signプラグインであれば、以下の様な記述で署名を作成することが可能です。

```groovy build.gradle
['maven', 'signing'].each {apply plugin : it}
artifacts {
    archives jar
}
signing {
    sign configurations.archives
}
```

しかし、maven-publishプラグインでは、明示的に署名ファイルも取り扱いたいので、

一工夫が必要になります。

```groovy build.gradle
// plugins see(1)
['maven-publish', 'signing'].each {apply plugin : it}
// zip sources. see(4)
task sourceJar(type : Jar) {
    from sourceSets.main.allJava
    // add classifier to jar file
    classifier = 'sources'
}
// zip javadocs. see(4)
task javadocJar(type : Jar, dependsOn : javadoc) {
    from javadoc.destinationDir
    // add classifier to jar file
    classifier = 'javadoc'
}
// collect artifacts. see(3)
artifacts { 
    archives jar
    archives sourceJar
    archives javadocJar
}
// sign task. see(2)
task signArchives (type : Sign, dependsOn : [jar, sourceJar, javadocJar]) {
    sign configurations.archives
}
// execute sign task. see(7)
task preparePublication (dependsOn : signArchives)
// extracting signature files with classifier and extension. see(5)
def getSignatureFiles = {
    def allFiles = tasks.signArchives.signatureFiles.collect{it}
    def signedSources = allFiles.find { it.name.contains('-sources') }
    def signedJavadoc = allFiles.find { it.name.contains('-javadoc') }
    def signedJar = (allFiles - [signedSources, signedJavadoc])[0]
    return [
            [archive: signedSources, classifier: 'sources', extension: 'jar.asc'],
            [archive: signedJavadoc, classifier: 'javadoc', extension: 'jar.asc'],
            [archive: signedJar,     classifier: null,      extension: 'jar.asc']
    ]
}
publishing {
    publications {
        // publishing artifacts
        jars(MavenPublication) {
            from components.java
            [
                    [jarTask : tasks.sourceJar,  classifier : 'sources', extension : 'jar'],
                    [jarTask : tasks.javadocJar, classifier : 'javadoc', extension : 'jar']                    
            ].each {archive ->
                artifact (archive.jarTask) {
                    classifier = archive.classifier
                    extension  = archive.classifier
                }
            }
        }
        // publishing signature files. see(6)
        jarSignatures (MavenPublication) {
            getSignatureFiles().each {signedArchive ->
                artifact (signedArchive.archive) {
                    classifier = signedArchive.classifier
                    extension  = signedArchive.extension
                }
            }
        }
        // publishing pom file
        pom(MavenPublication) {
            pom.withXml {
                def node = asNode()
                node.children().last() + {
                    resolveStrategy = Closure.DELEGATE_FIRST
                    name 'sample-project'
                    description 'give information of gradle maven-publish plugin'
                    url projectUrl
                    dependencies {
                        project.configurations.compile.dependencies.each {dep ->
                            dependency {
                                groupId dep.group
                                artifactId dep.name
                                version dep.version
                            }
                        }
                    }
                    licenses {
                        license {
                            name 'The Apache Software License, Version 2.0'
                            url 'http://www.apache.org/license/LICENSE-2.0.txt'
                            distribution 'repo'
                        }
                    }
                    scm {
                        url github
                        connection scmUrl
                        developerConnection developerUrl
                    }
                    developers {
                        developer {
                            id 'mike_neck'
                            name 'Shinya Mochida'
                            email 'mike <at> mikeneck.org'
                        }
                    }
                }
            }
        }
    }
    repositories {
        fladDirs "${project.projectDirs}/artifacts"
    }
}
```

変更点は次のとおりです。

#### (1) signingプラグインを導入します。

signingプラグインではmavenプラグインと連携して次のように署名を作成することができます。

```groovy build.gradle
artifacts {
    archives jar
    archives javadocJar
    archives sourceJar
}
signing {
    sign configurations.archives
}
```

しかし、mavenプラグインを使わないので、上記の方法では望みの署名ファイルを取得出来ません。

#### (2) `Sign`タイプのタスクを作成する

signingプラグインが入っているので、`type`が`Sign`のタスクを定義することができます。

このタスクを作成しておくと、指定したファイルに対して署名を作成することができます。

なお、このタスクは事前に署名対象のファイルがあることが前提なので、

`jar`、`javadocJar`、`sourceJar`タスクに依存しています。

#### (3) 成果物を一つの変数でアクセスできるようにする

`artifacts{}`ブロックでは指定した`configuration`に成果物を登録することができます。

次の例では`archives` configurationにjarタスク、javadocJarタスク、sourceJarタスクの成果を

登録します。

```groovy build.gradle
artifacts {
    archives jar
    archives javadocJar
    archives sourceJar
}
```

これによって、`configurations.archives`というプロパティから、

各種タスクの成果物にアクセスできるようになります。

#### (4) `Jar`タイプのタスクにclassifierを指定して、成果物のファイル名を修正します。

```groovy build.gradle
// zip sources
task sourceJar(type : Jar) {
    from sourceSets.main.allJava
    // add classifier to jar file
    classifier = 'sources'
}
```

これによって、作成されるsourcesJarのファイル名に`-sources`が含まれるようになります。

#### (5) 署名ファイルを取り出します

`type`が`Sign`のタスクの`getSignatureFiles()`メソッドは、署名したファイルのリストを返します。

それらを`classifier`によって、わけて取り出して、

改めて`classifier`と`extension`を付与します。

#### (6) 署名ファイルをそれぞれ発行します。

上記の(5)のクロージャー`getSignatureFiles`によって、

署名ファイルと`classifier`と`extension`を取得し、

それぞれartifactとして登録、発行します。

#### (7) 事前に実行しておくタスクをまとめたタスクを追加

署名ファイルを作成するタスクを確実に実行しておくために、

`preparePublication`タスクを作成します。

これを`publish`タスクの前に実行します。

---

それでは`publish`タスクを実行してみます。

```
$gradle clean pP publish
:clean
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:javadoc
:javadocJar
:sourceJar
:signArchives
:preparePublication
:generatePomFileForJarSignaturesPublication
:publishJarSignaturesPublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.jar.asc to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts
Transferring 0K from remote
Uploaded 0K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-sources.jar.asc to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts
Transferring 0K from remote
Uploaded 0K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-javadoc.jar.asc to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts
Transferring 0K from remote
Uploaded 0K
:generatePomFileForJarsPublication
:publishJarsPublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.jar to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts/
Transferring 1K from remote
Uploaded 1K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-sources.jar to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts/
Transferring 1K from remote
Uploaded 1K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-javadoc.jar to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts/
Transferring 33K from remote
Uploaded 33K
:generatePomFileForPomPublication
:publishPomPublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.pom to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts/
Transferring 1K from remote
Uploaded 1K
:publish

BUILD SUCCESSFUL

Total time: 10.894 secs
```

さて、署名ファイルが作成されているか確認します。

```
$ cd /Users/mike/IdeaProjects/sample-project/artifacts/org/mikeneck/sample/sample-project/1.0
$ ls | grep asc
sample-project-1.0-javadoc.jar.asc
sample-project-1.0-javadoc.jar.asc.md5
sample-project-1.0-javadoc.jar.asc.sha1
sample-project-1.0-sources.jar.asc
sample-project-1.0-sources.jar.asc.md5
sample-project-1.0-sources.jar.asc.sha1
sample-project-1.0.jar.asc
sample-project-1.0.jar.asc.md5
sample-project-1.0.jar.asc.sha1
```

それぞれ署名ファイルが作成されているようです。

では、署名ファイルを検証してみましょう。

```
$ gpg2 --verify sample-project-1.0-javadoc.jar.asc
gpg: Signature made 木  6/20 18:05:08 2013 JST using RSA key ID ABC12345
gpg: Good signature from "Shinya Mochida (Groovy/JavaScript Developer in Japan) <mike@mikeneck.org>"
$ gpg2 --verify sample-project-1.0-sources.jar.asc
gpg: Signature made 木  6/20 18:05:08 2013 JST using RSA key ID ABC12345
gpg: Good signature from "Shinya Mochida (Groovy/JavaScript Developer in Japan) <mike@mikeneck.org>"
$ gpg2 --verify sample-project-1.0.jar.asc
gpg: Signature made 木  6/20 18:05:08 2013 JST using RSA key ID ABC12345
gpg: Good signature from "Shinya Mochida (Groovy/JavaScript Developer in Japan) <mike@mikeneck.org>"
```

ちゃんと署名できていることが確認できました。


課題
---

さて、jarファイルの署名をすることは出来ました。

#### pom署名ファイル問題

しかし、残念なことにpomファイルの署名ができていません。

上述のpomファイルを変更するというところで記述した

`org.gradle.api.XmlProvider`の実装クラスは

`org.gradle.api.internal.xml.XmlTransformer.XmlProviderImpl`です。

そのクラスには`public void writeTo(java.io.File file)`というメソッドがあります。

そのメソッドを介してpomファイルを出力することが可能です。

したがって、次の手順でpomファイルの署名も発行することが可能ではないかと

考えられます。

1. pom出力タスク中でpomファイルを書き出し
1. pom出力タスク中で書きだしたpomファイルの署名をするタスクを実行
1. pomファイルの署名をするタスクから署名ファイルを取得
1. 署名ファイルをartifactとして発行


上記の手順を実行するようにビルドスクリプトを書いてみます。

以下、一部抜粋。

```groovy build.gradle
// pom file
ext {
    pomFilePath = "${project.projectDir}/tmp/pom.xml"
    pomFile = file(pomFilePath)
}
// task for signing pom
task signPom(type : Sign) {
    sign pomFile
}
// getting a signature of pom
def getPomSignatrure = {
    return project.tasks.signPom.signatureFiles.collect{it}[0]
}
publishing {
    publications {
        // publish pom
        pom(MavenPublication) {
            pom.withXml {
                def node = asNode()
                node.chidren().last() + {
                    dependencies {
                        resolveStrategy = Closure.DELEGATE_FIRST
                        project.configurations.compile.dependencies.each {dep ->
                            dependency {
                                groupId dep.group
                                artifactId dep.name
                                version dep.version
                            }
                        }
                    }
                }
                writeTo(project.ext.pomFile)
                project.tasks.signPom.execute()
                artifact (getPomSignature()) {
                    classifier = null
                    extension  = 'pom.asc'
                }
            }
        }
    }
}
```

ではpublishタスクを実行してみます。

```
$ gradle --daemon clean pP publish
:clean
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:javadoc
:javadocJar
:sourceJar
:signArchives
:preparePublication
:generatePomFileForJarSignaturesPublication
:publishJarSignaturesPublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.jar.asc to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts/
Transferring 0K from remote
Uploaded 0K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-sources.jar.asc to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts/
Transferring 0K from remote
Uploaded 0K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-javadoc.jar.asc to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts/
Transferring 0K from remote
Uploaded 0K
:generatePomFileForJarsPublication
:publishJarsPublicationToMavenRepository
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0.jar to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts/
Transferring 1K from remote
Uploaded 1K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-sources.jar to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts/
Transferring 1K from remote
Uploaded 1K
Uploading: org/mikeneck/sample/sample-project/1.0/sample-project-1.0-javadoc.jar to repository remote at file:/Users/mike/IdeaProjects/sample-project/artifacts/
Transferring 33K from remote
Uploaded 33K
:generatePomFileForPomPublication
:publishPomPublicationToMavenRepository FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':publishPomPublicationToMavenRepository'.
> Failed to publish publication 'pom' to repository 'maven'
   > Invalid publication 'pom': artifact file does not exist: '/Users/mike/IdeaProjects/sample-project/tmp/pom.xml.asc'

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 5.357 secs
```

`publishPomPublicationToMavenRepository`タスクで落ちてしまっています。

理由は署名ファイルが見つからないということです。

では、該当のディレクトリーの中身を見てみます。

```
$ ls temp
pom.xml
```

pomファイルだけしか出力されていません。

したがって、署名タスクが実行できてない状態になっているわけです。

実行されたタスクを挙げてみると、このようになっています。

+ :clean
+ :compileJava
+ :processResources UP-TO-DATE
+ :classes
+ :jar
+ :javadoc
+ :javadocJar
+ :sourceJar
+ :signArchives
+ :preparePublication
+ :generatePomFileForJarSignaturesPublication
+ :publishJarSignaturesPublicationToMavenRepository
+ :generatePomFileForJarsPublication
+ :publishJarsPublicationToMavenRepository
+ :generatePomFileForPomPublication
+ :publishPomPublicationToMavenRepository FAILED

よくみてみると、signPomタスクは実行されていません。

というわけで、明示的にsignPomタスクを実行する必要があるわけですが、

(`Task#execute()`で呼び出さないということ)

`Task#dependsOn`でsignPomタスクを指定しても、発行できないだけでなく、

実際にはpublishing-publicationのコンテキストでは`dependsOn`が使えません。

先ほどのビルドスクリプトを一部変更してみます。

```groovy build.gradle
publishing {
    publications {
        // 中略
        pom(MavenPublication) {
            dependsOn project.tasks.signPom
            pom.withXml {
                // 中略
                writeTo(project.ext.pomFile)
            }
            artifact (getPomSignature()) {
                classifier = null
                extension  = 'pom.asc'
            }
        }
    }
}
```

このビルドスクリプトをパースさせると次のようなエラーが発生します。

```
$ gradle tasks
Picked up _JAVA_OPTIONS: -Dfile.encoding=UTF-8

FAILURE: Build failed with an exception.

* Where:
Build file '/Users/mike/IdeaProjects/sample-project/build.gradle' line: 90

* What went wrong:
A problem occurred configuring root project 'sample-project'.
> Cannot create a Publication named 'dependsOn' because this container does not support creating elements by name alone. Please specify which subtype of Publication to create. Known subtypes are: MavenPublication

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 13.805 secs
```

たとえ、これがDSL上問題がなくても、

pomファイルの出力はsignPomタスクの後に実行されるので、

エラーが発生することは予見できます。

#### 現状考えられるpom署名ファイル回避方法

gradleではダイナミックにタスクの定義ができます。

これを利用して、pomファイルがない場合は、

pomファイルの生成を行い、

一時的なファイルに対して発行を行います。

そして、pomファイルが存在する場合には、

署名タスクを実行して、

artifact登録して、maven repositoryへ発行します。

整理すると…

+ maven publishプラグインを二回実行する。
+ pomファイルが存在しない場合は、signPomを実行しない、`withXml`で`writeTo`を使ってpomファイルを出力する。
+ pomファイルが存在する場合は、signPomを先に実行しておいて、署名ファイルもartifactとして発行する

ということになります。

ビルドスクリプトを以下に示します(該当部分のみ)。

```groovy build.gradle
// dynamic definition of preparePublication
if (pomFile.exists()) {
    task preparePublication (dependsOn : [signArchives, signPom])
} else {
    task preparePublication (dependsOn : signArchives)
}
// publishing if pomFile exists
publishing {
    publications {
        // 中略
        pom(MavenPublication) {
            dependsOn project.tasks.signPom
            pom.withXml {
                // 中略
                if (!project.ext.pomFile.exists()) {
                    writeTo(project.ext.pomFile)
                }
            }
            if (project.ext.pomFile.exists()) {
                artifact (getPomSignature()) {
                    classifier = null
                    extension  = 'pom.asc'
                }
            } else {
                delete(project.ext.pomFile)
            }
        }
    }
}
```

…で、これは実は失敗しました。

詳細はまた続きを書きます。


{% render_partial _includes/post/post_footer.html %}

