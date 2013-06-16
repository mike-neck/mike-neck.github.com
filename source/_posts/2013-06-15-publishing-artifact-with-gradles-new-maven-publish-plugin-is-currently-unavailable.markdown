---
layout: post
title: "gradle 1.4以降から追加されているmaven-publishプラグインでは、まだmaven central レポジトリーに上げられないようです。"
date: 2013-06-15 15:53
comments: true
categories: [gradle, groovy, java]
---

<img src="http://www.gradle.org/forum-assets/images/gradle_logo.gif"/>

こんにちわ、みけです。

あ、また、例によって記事が長いので、

結論だけ見たい人は前半部分だけを見て下さい。 - 大体5分以内。

で、gradleでのmaven centralへのライブラリー登録方法を知りたい方は中盤部分まで読んで下さい。 - トータル15分。

で、僕の強引なgradle遊びまで読みたい方は最後まで読むといいかもしれません。 - トータル30分。

---

**前半部分**


ここ数日gradle1.4以降についかされた**maven-publish**プラグインを使って

maven centralへのライブラリー登録方法を調べていましたが、

maven-publishプラグインでのmaven central repoへの登録はまだサポートされてません
===

ようです。

元記事はこちらです。

+ [How to publish artifacts signatures (.asc files) using maven-publish plugin?](http://forums.gradle.org/gradle/topics/how_to_publish_artifacts_signatures_asc_files_using_maven_publish_plugin)

以下、簡単な意訳＋要約文

> ### 質問
> 
> maven-publishプラグインを使ってレポジトリー情報の設定と、それと別個に、singningプラグインを使ってartifactsへのサインを行うことができますが、それらを連携させることができません。
> 
> どのようにすればmaven-publishプラグインで、artifactsとサインファイル(.ascファイル)をアップロードするようにできますか？
> 

> ### 回答
> 
> 今のところmaven-publishプラグインを使ってartifactとsignatureをアップロードすることはサポートされていません。**Gradle**に「.asc」ファイル(サインしたファイル)が通常のartifactではなく、特別なartifactであることを伝える手段がないことが問題となっています。
> 
> [こちらのロードマップ](https://github.com/gradle/gradle/blob/master/design-docs/publication-model.md)を参照下さい。なお、この機能に関する優先順位は高くありません。

> ### 再質問
> 
> 現在、古い方法でのアップロードはサポートされていますか？それとも手作業でやらないと駄目ですか？

> ### 回答
> 
> 古い方法でのアップロードは利用できます。

> ### 提案
> 
> ありがとうございました。
> 
> ところで、ドキュメントの方で新しいプラグインではmaven centralにアップロード出来ないという記述がないので追加してもらえますか？


とのことで、maven-publishプラグインでのmaven central repoへのアップロードはまだ対応されていないようです。これはこれで従来のmavenプラグインよりも便利なので、maven centralへのアップロードも可能になって欲しいとろこです。

---

**中盤部分**

なお、日本語でも情報は入手出来ますが、念の為にこちらにも記述しておきます。

現状のgradleを用いたmaven centralへのアップロード方法
===

概要は、山本裕介氏のこちらの記事を参照して下さい。

+ [【最新版】Maven Central Repository へのライブラリ登録方法 #maven](http://samuraism.jp/diary/2012/05/03/1336047480000.html)

また、gradleでの方法についてはこちらを参照して下さい。

+ [Automated Gradle project deployment to Sonatype OSS Repository(英語)](http://jedicoder.blogspot.jp/2011/11/automated-gradle-project-deployment-to.html)
+ [GradleでMaven Central Repositoryに成果物をリリースする(日本語)](http://d.hatena.ne.jp/int128/20130409/1365434513)


両方の記事に共通していますが、Sonatypeからmaven centralにアップロードする場合は、

+ PGPによる署名の作成
+ 求められた規約の順守

が求められます。


### なんでこんな面倒くさいのか？

maven centralにおいてあるライブラリーの品質や、pomなどの情報がバラバラで、一定の品質を保てなかったからのようです。

詳しくはこちらをお読み下さい(英語)。

+ [Improve The Central Repository and the Supporting the Maven Ecosystem](http://www.sonatype.com/people/2010/01/nexus-oss-ecosystem/)

central repositoryの品質向上のためにmaven central repoでは下記の規約を儲けています。

### pom.xmlへの要求事項

+ `<modelVersion>` - `4.0.0`を設定する
+ `<groupId>` - 申請するときにもgroupIdの申請が必要です。Sonatypeに申請するときはベースになるドメイン名で申請する必要があります。例えば、「`org.mikeneck`」は大丈夫ですが、「`mikeneck`」という**groupId**では駄目です。
+ `<artifactId>` - ライブラリーの名前ですね。
+ `<version>` - バージョン番号です。なお、`1.0.0-SNAPSHOT`というような`SNAPSHOT`というバージョンはSonatypeではmaven central repoには乗せてくれませんの、注意して下さい。
+ `<packaging>` - 基本的には`jar`です。
+ `<name>` - プロジェクトの名前です。
+ `<description>` - ライブラリーに関する情報です。何をしてくれるライブラリーであるかを記述します。
+ `<url>` - プロジェクトのurlを設定します。
+ `<licenses>` - ライセンスに関する情報を設定します。中には下記の情報が一つ以上入っていることが求められます。
1. `<license><name>` - ライセンス名(ApacheライセンスとかLGPLとか)
1. `<license><url>` - ライセンス条項のurlを記述します
1. `<license><distribution>` - `repo`を設定します。
+ `<scm><url>` - レポジトリーのurlを記述します。git-hubの場合はsshのアドレス設定します。bitbucketのMercurialの場合はレポジトリーをWebで見る場合のurlを記述します。
+ `<scm><connection>` - git-hubであればsshのところで得られるurlの先頭に`scm:git:`を加えます。bitbucketのMercurialの場合はWebで見る場合のアドレスの先頭に`scm:hg:`を付与します。
+ `<developers>` - 開発者情報を記入します。その中の構成は次のとおりです。
1. `<developer><id>` - 開発者のID。通名とかでも良いようです。僕の場合は`mike_neck`を記入します。
1. `<developer><name>` - 開発者の名前です。僕の場合は`Shinya Mochida`になります。
1. `<developer><email>` - 開発者のメールアドレスです。僕の場合は`mike <at> mikeneck.org`としています。

### 配布するファイルへの規約

+ `<packaging>`が`jar`の場合にはjarファイルにはjavaクラスが入っていること
+ 名前が`projectname-version-javadoc.jar`というjavadocのjarが入っていること
+ 名前が`projectname-version-sources.jar`というソースのjarが入っていること
+ すべてのartifact(`pom.xml`、`projectname-version.jar`、`projectname-version-javadoc.jar`、`projectname-version-sources.jar`)に対してPGPによる署名がなされていること
+ 公開鍵が公開鍵サーバーから取得可能になっていること

となっています。

また、何らかの理由でjavadocのjarやsourcesのjarが作られない場合でも、READMEというファイルを含んだjavadocのjarよsourcesのjarを作る必要があります。

### gradleでmaven centralにアップロードするためのbuild.gradle

では、サンプルのbuild.gradleをここに上げておきます。

```groovy build.gradle
['groovy', 'signing', 'maven', 'idea'].each {
    apply plugin : it
}
// project information
sourceCompatibility = jdkVersion
targetCompatibility = jdkVersion
group = 'org.jojo.sample'
version = '1.0'
// dependency management
repositories {
    mavenCentral ()
}
dependencies {
    compile 'org.codehaus.groovy:groovy-all:2.1.3'
    testCompile 'org.spockframework:spock-core:0.7-groovy-2.0'
}
// compile options
tasks.withType(Compile) {
    options.encoding = 'UTF-8'
}
// javadoc settings (making template locale en_US)
javadoc {
    options.locale = 'en_US'
}
// creating jars
task sourcesJar (type : Jar) {
    classifier = 'sources'
    from sourceSets.main.allSource
}
task javadocJar (type : Jar, dependsOn : javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}
// collect artifacts to be signed
artifacts {
    archives jar
    archives sourceJar
    archives javadocJar
}
// sign artifacts
signing {
    sign configurations.archives
}
// uploading artifacts
uploadArchives {
    repositories.mavenDeployer {
        beforeDeployment {MavenDeployment deployment ->
            signPom(deployment)
        }
        repository (url : sonatypeUrl) {
            authentication (
                    userName : sonatypeUsername,
                    password : sonatypePassword)
        }
        pom.project {
            name project.name
            packaging 'jar'
            description 'sample project'
            url projectUrl
            licenses {
                license {
                    name 'The Apache Software License, Version 2.0'
                    url 'http://www.apache.org/license/LICENSE-2.0.txt'
                    distribution 'repo'
                }
            }
            scm {
                url githubUrl
                connection "scm:git:${githubUrl}"
                developerConnection "scm:git:${githubUrl}"
            }
            developers {
                developer {
                    id 'jojo'
                    name 'Jonathan Joester'
                    email 'mike <at> mikeneck.org'
                }
            }
        }
    }
}
```

また、よく使いまわす変数については`gradle.properties`に書いておきます。

```bash gradle.properties
jdkVersion=1.7
projectUrl=https://github.com/mike-neck/mike-neck.github.com
github=git@github.com:mike-neck/mike-neck.github.com.git
```

また、署名関連の変数などについては`~/.gradle/gradle.properties`に書いておきます。

```bash gradle.properties
# siging information
signing.keyId=ABCD1234
signing.password=HOGEpassword00
signing.secretKeyRingFile=/Users/username/.gnupg/secring.gpg
# sonatype information
sonatypeUrl=https://oss.sonatype.org/service/local/staging/deploy/maven2/
sonatypeUsername=username
sonatypePassword=password
```

あとは、`gradle`コマンドで`uploadArchives`を記述すれば、

Sonatypeの方にアップロードされます。

```
$ gradle uploadArchives
```

なお、事前にSonatypeでの[JIRAでissueを登録しておくこと](http://goo.gl/XXfRl)や、

[Nexus UIで最終的なステージング操作](http://goo.gl/w9Exz)をする必要があります。

参考までに、僕が以前作ったissueのリンクを張っておきます。

[OSSRH-4119 request for creating repository for Graffiti-mike](https://issues.sonatype.org/browse/OSSRH-4119)

[<img src="//googledrive.com/host/0B4hhdHWLP7RRN0diTW1CSGptaHM" style="width : 520px;"/>](https://googledrive.com/host/0B4hhdHWLP7RRN0diTW1CSGptaHM)

中盤終わり

---

### gradleであそぶコーナー

**TBD**と書きたいのですが…

上記の古いgradleでのアーカイブアップロードの方法は、

やや難点があります。

```
$ gradle tasks
------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Assembles and tests this project.
buildDependents - Assembles and tests this project and all projects that depend on it.
buildNeeded - Assembles and tests this project and all projects it depends on.
clean - Deletes the build directory.
jar - Assembles a jar archive containing the main classes.

Documentation tasks
-------------------
groovydoc - Generates Groovydoc API documentation for the main source code.
javadoc - Generates Javadoc API documentation for the main source code.

Help tasks
----------
dependencies - Displays all dependencies declared in root project 'properties-builder'.
dependencyInsight - Displays the insight into a specific dependency in root project 'properties-builder'.
help - Displays a help message
projects - Displays the sub-projects of root project 'properties-builder'.
properties - Displays the properties of root project 'properties-builder'.
tasks - Displays the tasks runnable from root project 'properties-builder' (some of the displayed tasks may belong to subprojects).

IDE tasks
---------
cleanIdea - Cleans IDEA project files (IML, IPR)
idea - Generates IDEA project files (IML, IPR, IWS)

Upload tasks
------------
uploadArchives - Uploads all artifacts belonging to configuration ':archives'

Verification tasks
------------------
check - Runs all checks.
test - Runs the unit tests.

Other tasks
-----------
cleanIdeaWorkspace
install - Installs the 'archives' artifacts into the local Maven repository.
wrapper

Rules
-----
Pattern: build<ConfigurationName>: Assembles the artifacts of a configuration.
Pattern: upload<ConfigurationName>: Assembles and uploads the artifacts belonging to a configuration.
Pattern: clean<TaskName>: Cleans the output files of a task.

To see all tasks and more detail, run with --all.

BUILD SUCCESSFUL

Total time: 10.478 secs
```

uploadArchivesのところを見ると、

`archives` configurationに登録されているすべてのartifactsをアップロードする

と書かれています。

ただ、これだと、ひとつのアップロードしか書いていくことができないので、

非常に面倒です。

たとえば、こういった局面があります。

+ in-houseリポジトリーにも登録する
+ maven centralにも登録する
+ 異なるartifactsをアップロードする
+ dependenciesにトリッキーなことをしているので、dependenciesの記述を書き換えたい

こういったケースでは、

一つ一つ別々のタスクとして記述をしていかないとできない場合があります。

gradleは柔軟性も求めるツールなので、

これらの要望も吸収して

簡単な記述でできるように常に進化を遂げています。


それを満たす機能が、今回のテーマの**maven-publish**プラグインです。

**maven-publish**プラグインでは`archives` configuration以外の成果物も

柔軟に発行できますし、依存性を書き換えることもできます。

記述はこんな感じになります。

```groovy build.gradle
['groovy', 'maven-publish'].each {
    apply plugin : it
}
// 中略
publishing {
    publications {
        ourMavenServer(MavenPublication) {
            from components.java
            artifact sourceJar
            pom.withXml {
                def node = asNode()
                node.removeNode(node.dependencies[0])
                asNode().children().last() + {
                    resolveStrategy = CLosure.DELEGATE_FIRST
                    // writing additional pom elements with builder style
                    name 'our-subproject'
                    description 'our-subproject description'
                    url 'https://www.google.com'
                }
                // overwrite dependencies
                asNode().dependencies[0].replaceNode {
                    resolveStrategy = Closure.DELEGATE_FIRST
                    dependencies {
                        def dep = project.configurations.another.dependencies
                        dep.each {d ->
                            dependency {
                                groupId d.group
                                artifactId d.name
                                version d.version
                                scope 'compile'
                            }
                        }
                    }
                }
            }
        }
    }
    repositories {
        maven {
            name 'in-house'
            url 'https://repos.mycompany.com/nexus/service/local/staging/deploy/maven2/'
        }
    }
}
```

上記の例でやっていることは

+ ドキュメント読まないのでclassesとsourcesだけをアーカイブ化
+ ちろっとpom.xmlに情報を追加
+ dependenciesを書き換え

一般常識的に考えれば、dependenciesの書き換えはマズイと思われますが…

某有名なライブラリーのpom.xmlでありもしないartifactを参照しているライブラリーがあり、

`compile` configurationではプロジェクトのdependencyを指定するのではなく、

```
configurations {
    another
}
```

と異なるconfigurationを設定して、

プロジェクトのdependencyを設定する場合などがあります。

某有名ライブラリーとは**org.eclipse.jetty:jetty-server**というんですけどね…

---

さて、もう少し遊んでいるんですが、

それは別の記事にしますね。



{% render_partial _includes/post/post_footer.html %}

