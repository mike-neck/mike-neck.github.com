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

### 新機能や追加機能

##### より速いGradleビルド

Gradle1.7でのビルドはより速くなります。

+ dependency解決が速くなります(ほとんどのビルドで改善効果が現れます)
+ テスト実行が高速化されます(特にログを大量に出力しているようなGradleで顕著です)
+ ビルドスクリプトのコンパイルが速くなります(Gradle1.6に比べて75%の時間でできるようになります)
+ 並行実行モードが高速になります

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

##### C++プロジェクトサポートの改善 <small>incubating</small>

GradleはC++プロジェクトのサポートをしていました。これはGradleをネイティブコードプロジェクトのビルドツールとして最良のものにする改善になります。

+ スタティックライブラリーの作成、リンク作成
+ 異なるC++ツールチェインでも利用可能になります(Visual C++、GCCなど)
+ 異なるアーキテクチャー、ビルドタイプ、OSへの対応
+ Variantに基づいた依存性解決
+ その他もろもろ(詳しくは[こちら](https://github.com/gradle/gradle/blob/master/design-docs/continuous-delivery-for-c-plus-plus.md)を見て下さい(英語))

##### JCenter レポジトリーのサポート <small>incubating</small>

[Bintray's JCenter Repository](https://bintray.com/bintray/jcenter)からdependencyを取得できるようになります。`jcenter()`レポジトリーノーテーションにより利用可能です。JCenterはコミュニティリポジトリーで、[Bintray](https://bintray.com/)から無料で配布可能です。

```groovy
repositories {
    jcenter()
}
```

このスクリプトにより[http://jcenter.bintray.com](http://jcenter.bintray.com)がApache Maven repositoryと同様にリポジトリーリストに追加されます。

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

これによりテストタスクも他のレポート生成タスクとAPIレベルで同等になります。また、不要であればJUnit XMLファイルの生成しないことも可能です。

##### ビルドダッシュボードの改善 <small>incubating</small>

上記の変更(テストタスクに標準的なレポートインターフェースを採用)は言い換えると、テストレポートが[ビルドダッシュボード](http://www.gradle.org/docs/release-candidate/userguide/buildDashboard_plugin.html)に現れるということです。

なお、`buildDashbord`タスクは自動的にレポート系タスクと同時実行されます(Finalizerタスクの機能により実現されています)。



{% render_partial _includes/post/post_footer.html %}

