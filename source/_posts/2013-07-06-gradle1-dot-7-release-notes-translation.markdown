---
layout: post
title: "gradle1.7のリリースノート超意訳"
date: 2013-07-06 19:23
comments: true
categories: [groovy, gradle]
---

<img src="//googledrive.com/host/0B4hhdHWLP7RRZHRNTkNwZEYtdkE"/>

こんにちわ、みけです。

現在、gradle1.7-rc1が利用できます。

リリースノートの超意訳をどうぞ。

---

Gradle Release Notes
---

#### Version 1.7-rc-1

Gradle1.7になるとすごく速くなります。dependency解決とビルドスクリプトのコンパイルの改善をしました。Gradleユーザーみんなこの恩恵に預かれますが、でっかいプロジェクトだと、その効果はもっと顕著になります。パフォーマンスの改善とスケービリティがGradle1.7の主要なテーマになっています。

これらの改善点に加えて、Gradle1.7では面白い機能がついてきます。finalizer taskメカニズムによってタスクの結果がどうであれ、タスクの次に別のタスクを起動させることができるようになります。例えば、アプリケーションサーバーを起動するようなintegrationテスト(の失敗)の後にアプリケーションサーバーを終了させることができるようになります。コピーやアーカイブ生成時にファイルの重複をコントロールする機能が登場します。

Gradle1.7のBuild Setupプラグインの改善によりプロジェクトをテンプレートから生成する機能が利用可能になります。これにより新規プロジェクトの作成が簡単になります。

C++からネイティブバイナリーを作成する機能も進化します。ネイティブバイナリーの生成に関しては結構面倒な領域ですが、今後もGradleのこの分野での進化を期待して下さい。

Gradle1.7ではコア・テベロップチーム以外からのコントリビュートが多いのも特徴です。Gradle1.7に貢献してくださったデベロッパーの皆様に感謝しております。

Table Of Contents
---

### [新機能や追加機能](#c1)

+ [より速いGradleビルド](#c1_1)
+ [Finalizer tasks](#c1_2)
+ [C++プロジェクトサポートの改善](#c1_3)
+ [JCenter レポジトリーのサポート](#c1_4)
+ [パターン・ベース・ファイル・コピー設定](#c1_5)
+ [ファイル重複時のハンドリング機能](#c1_6)
+ [Gradle Wrapperはビルドスクリプトを特にいじらなくても利用可能になります](#c1_7)
+ [Javaライブラリーテンプレートプロジェクトの生成](#c1_8)
+ [publicationのカスタマイズ - new publishingプラグインにおける](#c1_9)
+ [複数のモジュールを一つのGradleプロジェクトから発行する](#c1_10)
+ [TestNGパラメーターがテストレポートに出力されます](#c1_11)
+ [テストタスクに標準的なレポートインターフェースを採用](#c1_12)
+ [ビルドダッシュボードの改善](#c1_13)
+ [JUnit XMLファイルでテストケースごとにテストの出力を表示するようになりました](#c1_14)
+ [ApplicationプラグインでJVMパラメーターを指定できるようになります](#c1_15)
+ [BndライブラリーのアップデートによりOSGiサポートが改善されました](#c1_16)

### [修正された問題](#c2)

### [非推奨となったもの](#c3)

+ [テストレポートプロパティ](#c3_1)

### [潜在的な互換性に関わる変更](#c4)

+ [インメモリーdependencyキャッシング](#c4_1)
+ [incubatingのJaCoCoプラグインの変更](#c4_2)
+ [incubatingのbuild-setupプラグインの変更](#c4_3)
+ [incubatingのivy-publishプラグインにおけるタスク名の変更](#c4_4)
+ [IvyPublicationのデフォルト`status`値が`integration`となり、`project.status`でなくなりました](#c4_5)
+ [C++サポートの大幅な変更](#c4_6)
+ [`ConfigureableReport`が`ConfigurableReport`に名称が変わりました](#c4_7)
+ [テストがない場合テストタスクがスキップされるようになりました](#c4_8)
+ [OSGiプラグインに使われているBndライブラリーがアップデートされました](#c4_9)

### [コントリビューター](#c5)

### [既知の問題](#c6)

---

<span id="c1"></span>
### 新機能や追加機能

<span id="c1_1"></span>
##### より速いGradleビルド

Gradle1.7でのビルドはより速くなります。

+ dependency解決が速くなります(ほとんどのビルドで改善効果が現れます)
+ テスト実行が高速化されます(特にログを大量に出力しているようなGradleで顕著です)
+ ビルドスクリプトのコンパイルが速くなります(Gradle1.6に比べて75%の時間でできるようになります)
+ 並行実行モードが高速になります

<span id="c1_2"></span>
##### Finalizer tasks <small>incubating</small>

Gradle1.7から新しいタスク実行ルールが導入され、タスク終了時に他のタスクを起動することができるようになります。この機能は[Marcin Erdmann](https://github.com/erdi)氏によるものです。

[Finalizer tasks](http://www.gradle.org/docs/release-candidate/userguide/more_about_tasks.html#N10F84)ではタスク終了時にそのタスクの結果にかかわらず別のタスクを起動します。

```groovy
configure([integTest1, integTest2]) {
    dependsOn startAppServer
    finalizedBy stopAppServer
}
```

この例では`integTest1`タスクと`integTest2`タスクの終了時に`stopAppServer`タスクを実行するように宣言されています。どちらか片方のタスクがビルドの最中に起動された場合もfinalizer taskが自動で実行されます。ビルドの最中に両方のタスクが起動された場合でも必ず両方のタスクの最後にfinalizer taskが実行されます。finalizer taskの`stopAppServer`はGradle実行時に起動タスクとして指定する必要はありません。

<span id="c1_3"></span>
##### C++プロジェクトサポートの改善 <small>incubating</small>

GradleはC++プロジェクトのサポートをしていました。これはGradleをネイティブコードプロジェクトのビルドツールとして最良のものにする改善になります。

+ スタティックライブラリーの作成、リンク作成
+ 異なるC++ツールチェインでも利用可能になります(Visual C++、GCCなど)
+ 異なるアーキテクチャー、ビルドタイプ、OSへの対応
+ Variantに基づいた依存性解決
+ その他もろもろ(詳しくは[こちら](https://github.com/gradle/gradle/blob/master/design-docs/continuous-delivery-for-c-plus-plus.md)を見て下さい(英語))

<span id="c1_4"></span>
##### JCenter レポジトリーのサポート <small>incubating</small>

[Bintray's JCenter Repository](https://bintray.com/bintray/jcenter)からdependencyを取得できるようになります。`jcenter()`レポジトリーノーテーションにより利用可能です。JCenterはコミュニティリポジトリーで、[Bintray](https://bintray.com/)から無料で配布可能です。

```groovy
repositories {
    jcenter()
}
```

このスクリプトにより[http://jcenter.bintray.com](http://jcenter.bintray.com)がApache Maven repositoryと同様にリポジトリーリストに追加されます。

<span id="c1_5"></span>
##### パターン・ベース・ファイル・コピー設定 <small>incubating</small>

Gradle1.7ではきめ細かい設定によりどのファイルがコピーされるべきかを定義することができます。設定方法は"Ant Patterns"のように指定出来ます。この機能は[Kyle Marhan](https://github.com/kylewm)氏によるものです。

GradleにはファイルコピーのAPI([CopySpec](http://www.gradle.org/docs/release-candidate/javadoc/org/gradle/api/file/CopySpec.html))があり、アーカイブすることもできます。この新しい機能はより強力なものとなります。

```groovy
task copyFiles(type : Copy) {
    from 'src/files'
    into "$buildDir/copied-files"
    // Replace the version number variable in only the text files
    filesMaching('**/*.txt') {
        expand version: '1.0'
    }
}
```

`fileMatching`メソッドではClosureを引数に取り、[`FileCopyDetails`](http://www.gradle.org/docs/release-candidate/javadoc/org/gradle/api/file/FileCopyDetails.html)オブジェクトの設定を行うことができます。これと反対の動作をする[`filesNotMatching`](http://www.gradle.org/docs/release-candidate/javadoc/org/gradle/api/file/CopySpec.html#filesNotMatching%28java.lang.String%2C%20org.gradle.api.Action%29)というメソッドもあり、パターンに該当しないファイルすべてを指定することもできます。

<span id="c1_6"></span>
##### ファイル重複時のハンドリング機能 <small>incubating</small>

ファイルのコピーやアーカイブするときに、よく重複することがあります。そのような重複が発生した場合の処理方法を設定できるようになります。

```groovy
task zip(type : Zip) {
    from 'dir1'
    from 'dir2'
    duplicatesStrategy 'exclude'
}
```

処理方法には２つあります。`include`と`exclude`です。

`include`ストラテジーの場合、既存のGradleの動作と変わりありません。後からコピーされたもので上書きされます。なお、実際に発生した場合には警告が出力されるようになります。アーカイブ(zip、jar)の場合は新しいものが作成されます。

`exclude`ストラテジーの場合、重複したファイルは無視されます。最初にコピーされたファイルが最終的に使われ、その後のファイルは無視され続けます。アーカイブの場合も同様で、最初に作成されたら、同じファイル名のものは作成されません。

```groovy
task zip (type : Zip) {
    duplicatesStrategy 'exclude' // default strategy
    from ('dir1') {
        filesMatching('**/*.xml') {
            duplicatesStrategy 'include'
        }
    }
    from ('dir2') {
        duplicatesStrategy 'include'
    }
}
```

<span id="c1_7"></span>
##### Gradle Wrapperはビルドスクリプトを特にいじらなくても利用可能になります <small>incubating</small>

[Gradle Wrapper](http://www.gradle.org/docs/release-candidate/userguide/gradle_wrapper.html)は`Wrapper`タスクを定義しなくても利用できるようになります。つまり`Wrapper`のためにビルドスクリプトをいじることはありません。

`Wrapper`を利用するためには単純に次のコマンドを実行するだけです。

```
$ gradle wrapper
```

`Wrapper`ファイルがこれにより生成・設定されて、現在使用しているGradleのバージョンを利用することができるようになります。

なお、`wrapper`タスクをカスタマイズしたい場合は、ビルドスクリプトを次のように変更します。

```groovy
wrapper {
    gradleVersion '1.6'
}
```

もし既存の`Wrapper`タイプのタスクがある場合は、そちらが使われます。それ以外の場合はデフォルトの`wrapper`タスクが使用されます。

<span id="c1_8"></span>
##### Javaライブラリーテンプレートプロジェクトの生成

build-setupプラグインがプロジェクトタイプをサポートするようになりました。Gradle1.7ではjava-libraryタイプが利用可能です。このタイプで生成されるのは以下のとおりです。

+ javaプラグインが適用されたシンプルなビルドファイル
+ サンプルのプロダクションクラスとディレクトリー
+ サンプルのJUnitテストとディレクトリー
+ Gradle Wrapperファイル

Javaライブラリープロジェクトを作る場合は次のコマンドを実行するだけです(build.gradleファイルは必要ありません)。

```
$ gradle setupBuild --type java-library
```

詳しくはこちらを[Build Setup plugin](http://www.gradle.org/docs/release-candidate/userguide/build_setup_plugin.html)を参照して下さい。

<span id="c1_9"></span>
##### publicationのカスタマイズ - new publishingプラグインにおける <small>incubating</small>

publishプラグインでアーカイブを発行するときにartifactの情報を明示的にカスタマイズできるようになりました。以前はartifactの情報はプロジェクトの情報から取得されていました。

`MavenPublication`では`groupId`、`artifactId`と`version`情報を設定できるようになります。Mavenの`pom`の`packaging`の値も設定可能です。

```groovy
publications {
    mavenPub(MaenPublication) {
        from components.java
        groupId 'my.group.id'
        artifactId 'my-publication'
        version '3.1'
        pom.packaging 'pom'
    }
}
```

`IvyPublication`では`organisation`、`module`と`revision`を設定出来ます。`IvyModuleDescriptor`で`status`の値も設定可能です。

```groovy
publications {
    ivyPub(IvyPublication) {
        from components.java
        organisation 'my.org'
        module 'my-module'
        revision '3'
        descriptor.status 'milestone'
    }
}
```

この機能のすごいところは、`module`や`artifactId`を設定することができることです。というのも、これまでは`project.name`を使っており、Gradleのビルドスクリプトで変更することができなかったためです。

<span id="c1_10"></span>
##### 複数のモジュールを一つのGradleプロジェクトから発行する <small>incubating</small>

publishプラグインでは複数のモジュールを一つのGradleプロジェクトから発行することが可能になります。

```groovy
project.group 'org.cool.library'
publications {
    implJar(MavenPublication) {
        artifactId 'cool-library'
        version '3.1'
        artifact jar
    }
    apiJar(MavenPublication) {
        artifactId 'cool-library-api'
        version '3'
        artifact apiJar
    }
}
```

これまでも可能でしたが、ここに示したような簡単な方法ではできませんでした。新しい`ivy-publish`、`maven-publish`プラグインでは簡単に出来ます。

<span id="c1_11"></span>
##### TestNGパラメーターがテストレポートに出力されます <small>incubating</small>

TestNGでは[parameterizing test methods](http://testng.org/doc/documentation-main.html#parameters)によってあるテストを複数回、異なるデータの入力でテストを行うことがサポートされています。以前のGradleのレポートでは、パラメタライズドテストは複数行にわたってレポートが出力されていましたが、そのテストの区別をすることができませんでした。テストレポートにはパラメーターの`toString()`メソッドによる値を出力できるようにし、どのデータによるテストかを区別できるようにしました。

```java ParameterizedTest.java
import org.testng.annotations.*;

public class ParameterizedTest {
    @Test(dataProvider = "1")
    public void aParameterizedTestCase(String var1, String var2) {
        // do testing
    }

    @DataProvider(name = "1")
    public Object[][] provider1() {
        return new Object[][] {
           {"1", "2"},
           {"3", "4"}
        };
    }
}
```

上記のテストに対するレポートは次のように表示されます。

+ `aParameterizedTestCase(1, 2)`
+ `aParameterizedTestCase(3, 4)`

この情報はGradleのHTMLテストレポートとJUnit XMLファイルに反映されます。JUnit XMLファイルは一般的にテストの実行結果をCIサーバーに引き渡す役割を担っており、CIサーバーでパラメーターの情報を参照することが可能になります。

<span id="c1_12"></span>
##### テストタスクに標準的なレポートインターフェースを採用

レポートインターフェースによりレポートをコントロールする方法が提供されます。テストタスクではこのレポートインターフェースが採用されます。

```groovy
apply plugin: 'java'
test {
    reports {
        html.enabled = false
        junitXml.destination = file("$buildDir/junit-xml")
    }
}
```

testタスクの提供する[TestReports](http://www.gradle.org/docs/release-candidate/javadoc/org/gradle/api/tasks/testing/TestReports.html)型の[`ReportContainer`](http://www.gradle.org/docs/release-candidate/javadoc/org/gradle/api/reporting/ReportContainer.html)を通じて、HTMLによるテストレポートおよびJUnit XMLファイルの出力の制御を行えるようになります(これらのファイルは通常CIサーバーやその他のツールとテスト結果を共有するために使われます)。

これによりテストタスクも他のレポート生成タスクとAPIレベルで同等になります。また、不要であればJUnit XMLファイルを生成しないことも可能です。

<span id="c1_13"></span>
##### ビルドダッシュボードの改善 <small>incubating</small>

上記の変更(テストタスクに標準的なレポートインターフェースを採用)は言い換えると、テストレポートが[ビルドダッシュボード](http://www.gradle.org/docs/release-candidate/userguide/buildDashboard_plugin.html)に現れるということです。

なお、`buildDashbord`タスクは自動的にレポート系タスクと同時実行されます(Finalizerタスクの機能により実現されています)。


<span id="c1_14"></span>
##### JUnit XMLファイルでテストケースごとにテストの出力を表示するようになりました <small>incubating</small>

この変更により[Jenkins](http://jenkins-ci.org/)などのCIサーバーでよりよいテスト結果を得ることができます。

JUnit XMLファイルはテスト結果のフォーマットとしてデファクト・スタンダードです。たいていのCIサーバーはこのファイルをテストの実行結果として使っています。元々は「JUnit And Tasks」によって考えられたもので、JUnitとほぼ同時期に作られ、多くの局面で使われてきましたが、特に仕様が定められていませんでした。

このファイルにはテスト中の標準出力(`System.out`と`System.err`)の内容が収められています。これまでは出力は_クラスレベル_でのみ記録されていました。つまり、テストケース毎の出力が得られなかったわけです。しかしGradleでは「テストケース毎の出力」モードを利用することができるようになりました。

```groovy
test {
    reports {
        junitXml.outputPerTestCase = true
    }
}
```

このモードを使うと、XMLレポートはテストケースごとに作成されます。Jenkins CIサーバーではテストケース毎の結果を参照できるようになります。`outputPerTestCase = true`と設定しておくと、テストケース毎の出力が画面に表示されます。以前は_テストクラス_毎の出力でした。

Jenkinsの[JUnit Attachments Plugin](https://wiki.jenkins-ci.org/display/JENKINS/JUnit+Attachments+Plugin)はこの機能とともに用いることで有効活用できます。

<span id="c1_15"></span>
##### ApplicationプラグインでJVMパラメーターを指定できるようになります <small>incubating</small>

[Olaf Kilschat](https://github.com/multi-io)により[Application Plugin](http://www.gradle.org/docs/release-candidate/userguide/application_plugin.html)はデフォルトのJVM引数を設定することができるようになります。

```groovy
apply plugin: 'application'
applicationDefaultJvmArgs = ['-Dfile.encoding=UTF-8']
```

<span id="c1_16"></span>
##### BndライブラリーのアップデートによりOSGiサポートが改善されました

[OSGi](http://www.gradle.org/docs/release-candidate/userguide/osgi_plugin.html)プラグインは[Bnd](http://www.aqute.biz/Bnd/Bnd)ツールを使ってbundle manifestsを作成しています。Bndツールのバージョンが1.50.0から2.1.0に変更されます。

最も重要な変更点は"invokedynamic"命令を使うJavaコード用のmanifestが正確になることです。

<span id="c2"></span>
### 修正された問題

Gradle1.7で30の問題が修正されました。

+ [[GRADLE-642]](http://issues.gradle.org/browse/GRADLE-642) - scala toolsまたはライブラリーが登録されていない場合にscalaタスクから出力されるエラーメッセージを修正
+ [[GRADLE-1289]](http://issues.gradle.org/browse/GRADLE-1289) - "create-project"コマンドによるスケルトンプロジェクト作成機能を提供
+ [[GRADLE-1372]](http://issues.gradle.org/browse/GRADLE-1372) - wrapperタスクをビルトインタスクにしました
+ [[GRADLE-1387]](http://issues.gradle.org/browse/GRADLE-1387) - Gradle Architypes機能を提供
+ [[GRADLE-1456]](http://issues.gradle.org/browse/GRADLE-1456) - Application PluginにてJAVA_OPTS/APPL_OPTSを設定する方法を提供
+ [[GRADLE-1551]](http://issues.gradle.org/browse/GRADLE-1551) - http://www.gradle.org/build_lifecycle.htmlのtypoを修正
+ [[GRADLE-1583]](http://issues.gradle.org/browse/GRADLE-1583) - Gradleがivy configurationの表現をサポートしていない
+ [[GRADLE-1704]](http://issues.gradle.org/browse/GRADLE-1704) - "gradle --version"コマンドの日付フォーマットの修正
+ [[GRADLE-1742]](http://issues.gradle.org/browse/GRADLE-1742) - CodeNarcの警告数の上限を設定できるようにした
+ [[GRADLE-2171]](http://issues.gradle.org/browse/GRADLE-2171) - zipファイル生成する場合に重複を回避するオプションを提供
+ [[GRADLE-2519]](http://issues.gradle.org/browse/GRADLE-2519) - `testReport = false`と指定していてもテストFAILURE時に存在しないファイルのURLが表示される
+ [[GRADLE-2666]](http://issues.gradle.org/browse/GRADLE-2666) - maven2GradleでNullpointerが発生
+ [[GRADLE-2702]](http://issues.gradle.org/browse/GRADLE-2702) - testRuntime/testCompile configurationがテストがなくても解決される
+ [[GRADLE-2738]](http://issues.gradle.org/browse/GRADLE-2738) - dependenciesで同じartifactの複数のバージョンを指定している場合、一番古いバージョンで解決される
+ [[GRADLE-2752]](http://issues.gradle.org/browse/GRADLE-2752) - 自分自身のライブラリーの古いバージョンに依存している場合に
+ [[GRADLE-2760]](http://issues.gradle.org/browse/GRADLE-2760) - `--parallel-threads`が内部的に4つまでしか使えない
+ [[GRADLE-2765]](http://issues.gradle.org/browse/GRADLE-2765) - XMLテストレポートを出力できない設定を追加
+ [[GRADLE-2766]](http://issues.gradle.org/browse/GRADLE-2766) - Ivy defaultConfMappingが解決に用いられない
+ [[GRADLE-2780]](http://issues.gradle.org/browse/GRADLE-2780) - Gradle1.6でスクリプトのコンパイルが劣化
+ [[GRADLE-2790]](http://issues.gradle.org/browse/GRADLE-2790) - Ivy dependency configuration mapping `'*->@'` がターゲットモジュールからすべてのconfigurationを含んでしまう
+ [[GRADLE-2791]](http://issues.gradle.org/browse/GRADLE-2791) - Ivy dependency configuration mapping で左項`'%'`が無視される
+ [[GRADLE-2792]](http://issues.gradle.org/browse/GRADLE-2792) - Ivy dependency configuration mapping で左項`'*,!A'`がAの解決時に含まれてしまう
+ [[GRADLE-2793]](http://issues.gradle.org/browse/GRADLE-2793) - Ivy dependency configuration mapping で右項`'#'`が不適切なターゲットconfigurationに解される
+ [[GRADLE-2802]](http://issues.gradle.org/browse/GRADLE-2802) - Wrapperのインストール/ダウンロードが`-g`、`--gradle-user-home`コマンドラインフラグを考慮しない
+ [[GRADLE-2806]](http://issues.gradle.org/browse/GRADLE-2806) - OSGiプラグインにBNDライブラリーの最新版を適用
+ [[GRADLE-2808]](http://issues.gradle.org/browse/GRADLE-2808) - `setting options.forkOptions.executable`と設定をしてjoint compileすると`NotSerializableException`が発生する
+ [[GRADLE-2813]](http://issues.gradle.org/browse/GRADLE-2813) - `project.exec()`でタスクを定義した場合、DSLが反映されない
+ [[GRADLE-2815]](http://issues.gradle.org/browse/GRADLE-2815) - `@Input`が`boolean`型のgetterで`is*`という名前のメソッドには適用されない
+ [[GRADLE-2821]](http://issues.gradle.org/browse/GRADLE-2821) - テストのないテストタスクのレポート出力時にTestReportタスクがエラーを出力
+ [[GRADLE-2825]](http://issues.gradle.org/browse/GRADLE-2825) - `strategy`が`include`に設定されていてもファイル重複の警告が出力される

<span id="c3"></span>
### 非推奨となったもの

<span id="c3_1"></span>
##### テストレポートプロパティ

<span id="c4"></span>
### 潜在的な互換性に関わる変更

<span id="c4_1"></span>
##### インメモリーdependencyキャッシング

<span id="c4_2"></span>
##### incubatingのJaCoCoプラグインの変更

<span id="c4_3"></span>
##### incubatingのbuild-setupプラグインの変更

<span id="c4_4"></span>
##### incubatingのivy-publishプラグインにおけるタスク名の変更

<span id="c4_5"></span>
##### IvyPublicationのデフォルト`status`値が`integration`となり、`project.status`でなくなりました

<span id="c4_6"></span>
##### C++サポートの大幅な変更

<span id="c4_7"></span>
##### `ConfigureableReport`が`ConfigurableReport`に名称が変わりました

<span id="c4_8"></span>
##### テストがない場合テストタスクがスキップされるようになりました

<span id="c4_9"></span>
##### OSGiプラグインに使われているBndライブラリーがアップデートされました

<span id="c5"></span>
### コントリビューター

Gradleコミュニティの他に、Gradleのディベロップメントチームは下記の人達にこのバージョンのGradleへの貢献に感謝します。

##### [Marchin Erdmann](https://github.com/erdi)

+ Finalizer tasks

##### [Dan Stine](https://github.com/dstine)

+ CodeNrcプラグインのmaxPriorityViolations setting ([GRADLE-1742](http://issues.gradle.org/browse/GRADLE-1742))
+ ユーザーガイドの修正

##### [Kyle Mahan](https://github.com/kylewm)

+ アーカイブ・コピー操作における重複への対処([GRADLE-2171](http://issues.gradle.org/browse/GRADLE-2171))
+ パターン・ベース・ファイル・コピー設定

##### [Robert Kühne](https://github.com/sponiro)

+ ユーザーガイドのスペルミス修正

##### [Björn Kautler](https://github.com/Vampire)

+ Build Dashboard のサンプルの修正

##### [Seth Goings](https://github.com/sgoings)

+ ユーザーガイドの修正

##### [Scott Bennett-McLeish](https://github.com/sbennettmcleish)

+ ユーザーガイドの修正

##### [Wujek Srujek](https://github.com/wujek-srujek)

+ wrapperのインストールロケーションに関する-gコマンドラインオプションの処理([GRADLE-2802](http://issues.gradle.org/browse/GRADLE-2802))

##### [Guillaume Laforge](https://github.com/glaforge)

+ OSGiプラグインのBndライブラリーアップデート([GRADLE-2806](http://issues.gradle.org/browse/GRADLE-2806))

##### [Yoav Landman](https://github.com/yoav)

+ `jcenter()`メソッドによるレポジトリー定義([Bintray's JCenter repository](https://bintray.com/bintray/jcenter))

<span id="c6"></span>
### 既知の問題

今のところGradle1.7に問題は見つかっていません。

{% render_partial _includes/post/post_footer.html %}

