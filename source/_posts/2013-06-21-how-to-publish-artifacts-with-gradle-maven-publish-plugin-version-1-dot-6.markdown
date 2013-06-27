---
layout: post
title: "How to publish artifacts to maven central repository via gradle maven-publish plugin (version 1.6)"
date: 2013-06-21 14:04
comments: true
categories: [groovy, java, gradle]
---

<img src="http://www.gradle.org/forum-assets/images/gradle_logo.gif"/>

Gradle **maven-publish plugin** provides the easier way to publish artifacts than the old **maven plugin**.

This post introduces you the way to publish artifacts with **maven-publish plugin**.

**Please Note that** maven-publish plugin is incubating feature. Its DSL may change later.


Goal
---

After reading this post, you can upload your artifacts to maven central repository via **maven-publish plugin**.


Basics and Example
---

To publish artifacts you should do these things.

1. to declare applying **maven-publish plugin**.
1. to tell gradle which files should be published.
1. to tell gradle where to upload artifacts.

Now let's see sample build script.

```groovy build.gradle
// declaration of plugins (1)
['java', 'maven-publish'].each {
    apply plugin : it
}
group = 'com.yourdomain'
version = '1.0'
repositories {
    mavenCentral ()
}
dependencies {
    compile 'org.apache.commons:commons-lang3:3.1'
    testCompile 'junit:junit:4.11'
}
publishing {
    publications {
        myPublication(MavenPublication) {
            // telling gradle to publish project's jar archive (2)
            from components.java
            // telling gradle to publish README file (2)
            artifact ('README.txt') {
                classifier = 'README'
                extension  = 'txt'
            }
        }
    }
    // telling gradle to publish artifact to local directory (3)
    repositories {
        maven {
            url "file:/${project.projectDir}/artifacts"
        }
    }
}
```

With this script you can publish your artifact via this command.

```
$ gradle publish
```

Then you will find some file is generated at `artifact` directory.

These files are …

+ sample-project.jar
+ sample-project.jar.md5
+ sample-project.jar.sha1
+ sample-project.pom
+ sample-project.pom.md5
+ sample-project.pom.sha1
+ sample-project-README.txt
+ sample-project-README.txt.md5
+ sample-project-README.txt.sha1


Conventions
---

**maven-publish plugin** has some conventions.

+ base archive name is project name.
+ `classifier` is given after the project name.
+ `extension` is given after the project name and `classifier`.
+ `classifier` and `extension` should be unique in all artifacts in a publication.


Publishing javadoc and source code as jar
---

### javadoc

Following shows the way to publish javadoc as jar.

1. call javadoc task.
1. create a task of zipping javadoc and call it.
1. give th zipping task to `artifact` method in publication container.


```groovy build.gradle
// (2)
task javadocJar (type: Jar, dependsOn: javadoc) { // (1)
    classifier = 'javadoc'
    from javadoc.destinationDir
}
publishing {
    publications {
        myPublication(MavenPublication) {
            artifact (javadocJar) { // (3)
                classifier = 'javadoc'
            }
        }
    }
}
```

### source codes

Following shows the way to publish source as jar.

1. create a task of zipping sources as jar.
1. give the zipping task to `artifact` method in publication container.

```groovy build.gradle
// (1)
task sourceJar (type : Jar) {
    classifier = 'sources'
    from sourceSets.main.allSource
}
publishing {
    publications {
        myPublication(MavenPublication) {
            artifact (sourceJar) { // (2)
                classifier = 'sources'
            }
        }
    }
}
```


Modifying POM
---

### requirements

**maven-publish plugin** generates POM, but it lacks some elements required by Sonatype OSS repository. Folowing shows list of elements to be added.

+ `<name>` - the name of project
+ `<description>` - the description for the project
+ `<url>` - project's url
+ `<scm><url>` - repository url.
+ `<scm><connection>` - repository url for scm tool. for example using git - github, it becomes `scm:git:git://github.com/your-name/project-name.git`
+ `<scm><developerConnection>` - repository url for scm tool via ssh. for example using git - github, it becomes `scm:git:ssh:git@github.com:your-name/project-name.git`
+ `<licenses><license><name>` - license name (i.e. `The Apache Software License, Version 2.0` etc…). In the case of the project being licensed under multiple license, `licenses` elements can have multiple `<license>` elements.
+ `<licenses><license><url>` - license url (e.x. if the project is licensed under Apache version 2, it becomes `http://www.apache.org/license/LICENSE-2.0.txt`)
+ `<licenses><license><distribution>` - `repo`
+ `<developers><developer><id>` - developer's id. If there are more than one developers, you can write `<developer>` elements more than one times.
+ `<developers><developer><name>` - developer's name.
+ `<developers><developer><email>` - developer's email.


### build script

To add these elements to POM, you can acces pom file via `pom` object's `withXml` method in `MavenPublication` container.

```groovy build.gradle
publishing {
    publications {
        myPublication (MavenPublication) {
            from components.java
            pom.withXml {
                asNode().children().last() + {
                    resolveStrategy = Closure.DELEGATE_FIRST
                    name 'project-name'
                    description 'description for project'
                    url projectUrl
                    scm {
                        url scmUrl
                        connection connectionUrl
                        developerConnection developerConnectionUrl
                    }
                    licenses {
                        license {
                            name 'The Apache Software License, Version 2.0'
                            url 'http://www.apache.org/license/LICENSE-2.0.txt'
                            distribution 'repo'
                        }
                    }
                    developers {
                        developer {
                            id 'your id or nick name'
                            name 'Your Name'
                            email 'your@mail.address'
                        }
                    }
                }
            }
        }
    }
}
```


Signing Jar
---

To keep quality of maven central repo, signing files is required.

These files should be signed.

+ main jar (file name is `project-name.jar.asc`)
+ javadoc jar (file name is `project-name-javadoc.jar.asc`)
+ sources jar (file name is `project-name-sources.jar.asc`)
+ pom file (file name is `project-name.pom.asc`, on this signature this post will mention later)

To sign archives is available via **signing plugin**.

```groovy build.gradle
// adding 'signing' plugin
apply plugin: 'signing'
// summarize artifacts
artifacts {
    archives jar
    archives sourceJar
    archives javadocJar
}
// sign all artifacts
task signJars (type : Sign, dependsOn: [jar, javadocJar, sourceJar]) {
    sign configurations.archives
}
// call signJar task before publish task
task preparePublish(dependsOn: signJar)
// extract signature file and give them proper name
def getSignatureFiles = {
    def allFiles = project.tasks.signJars.signatureFiles.collect { it }
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
        signatures (MavenPublication) {
            // give signature files to rtifact method
            getSignatureFiles().each {signature ->
                artifact (signature.archive) {
                    classifier = signature.classifier
                    extension  = signature.extension
                }
            }
        }
    }
}
```


Signing POM
---

Before running `publish` task, there are no POM file, so calling signing POM task will fail. To avoid this, whether calling POM task or not is defined dynamicly. And writing POM is available `writeTo(File)` method via `XmlProviderContainer` (i.e. on the `Closure` block of `pom.withXml`)

```groovy build.gradle
ext {
    pomFilePath = "${project.projectDir}/tmp/pom.xml"
    pomFile = file(pomFilePath)
}
configurations {
    pom
}
artifacts {
    archives jar
    archives sourceJar
    archives javadocJar
    if (pomFile.exists()) {
        pom pomFile
    }
}
task signPom(type: Sign) {
    sign configurations.pom
}
def getPomSignature = {
    return project.tasks.signPom.signatureFiles.collect{it}[0]
}
if (project.ext.pomFile.exists()) {
    task preparePublication (dependsOn : [signJars, signPom])
} else {
    task preparePublication (dependsOn : signJars)
}
publishing {
    publications {
        jar(MavenPublication) {
            // publishing main jars
            pom.withXml {
                // add required elements
                // here writing pom file
                if (!project.ext.pomFile.exists()) {
                    writeTo (project.ext.pomFile)
                }
            }
        }
        gpgJars(MavenPublication) {
            // publishing signature of jars
        }
        // dynamic publication definition
        // pom file does exist signature of pom file is published
        if (project.ext.pomFile.exists()) {
            gpgPom(MavenPublication) {
                artifact (getPomSignature()) {
                    classifier = null
                    extension  = 'pom.asc'
                }
            }
        }
    }
    repositories {
        maven {
            if (project.ext.pomFile.exists()) {
                url sonatypeUrl
                credentials {
                    username = sonatypeUsername
                    password = sonatypePassword
                }
            } else {
                url fileDirectory
            }
        }
    }
}
```

and execute gradle tasks as follows

```
$ gradle clean pP publish
$ gradle clean pP publish
```

You should execute gradle publish task twice.

1. The first execution is generating pom file and publishing som artifacts to machine's directory.
1. The second execution is publishing pom signature to Sonatype OSS repository.

#### Please note…

`publish` task will execute publication tasks according to the alphabetiacl order of publishing task name. And each publication task will generate POM file. So please take care of publication name. The recomending name for publications is …

+ gpgJars - publish signatures of jar files.
+ gpgPom - publish signature of POM.
+ jar - publish all jars and POM.


Credential
---

You may know an account of Sonatype OSS is required to upload artifact into maven central repo. Here shows settings of sonatype account in **maven-publish plugin**.

```groovy build.gradle
publishing {
    repositories {
        url 'https://oss.sonatype.org/service/local/staging/deploy/maven2/'
        credentials {
            username = sonatypeUsername
            password = sonatypePassword
        }
    }
}
```


Conclusion
---

Taking these things in account, here is a perfect example script for publishing artifacts to maven central repo with **maven-publish plugin**.

```groovy build.gradle
['java', 'siging', 'maven-publish'].each {
    apply plguin: it
}
// project information
group = 'com.yourdomain'
version = '1.0'
// dependency management as you like
repositories {
    mavenCentral ()
}
dependencies {
    compile 'org.apache.commons:commons-lang3:3.1'
    testCompile 'junit:junit:4.11'
}
// javadoc.jar generation
task javadocJar (type: Jar, dependsOn: javadoc) { // (1)
    classifier = 'javadoc'
    from javadoc.destinationDir
}
// sources.jar generation
task sourceJar (type : Jar) {
    classifier = 'sources'
    from sourceSets.main.allSource
}
// pom file name
ext {
    pomFilePath = "${project.projectDir}/tmp/pom.xml"
    pomFile = file(pomFilePath)
}
// add configuration for pom signing
configurations {
    pom
}
// summarize artifacts
artifacts {
    archives jar
    archives sourceJar
    archives javadocJar
    if (pomFile.exists()) {
        pom pomFile
    }
}
// sign all artifacts
task signJars (type : Sign, dependsOn: [jar, javadocJar, sourceJar]) {
    sign configurations.archives
}
// sign pom
task signPom(type: Sign) {
    sign configurations.pom
}
// defining which tasks should be called
if (project.ext.pomFile.exists()) {
    task preparePublication (dependsOn : [signJars, signPom])
} else {
    task preparePublication (dependsOn : signJars)
}
// extract signatures and add classifier and extension to them
def getSignatureFiles = {
    def allFiles = project.tasks.signJars.signatureFiles.collect { it }
    def signedSources = allFiles.find { it.name.contains('-sources') }
    def signedJavadoc = allFiles.find { it.name.contains('-javadoc') }
    def signedJar = (allFiles - [signedSources, signedJavadoc])[0]
    return [
            [archive: signedSources, classifier: 'sources', extension: 'jar.asc'],
            [archive: signedJavadoc, classifier: 'javadoc', extension: 'jar.asc'],
            [archive: signedJar,     classifier: null,      extension: 'jar.asc']
    ]
}
// extract pom signature
def getPomSignature = {
    return project.tasks.signPom.signatureFiles.collect{it}[0]
}
publishing {
    publicaitons {
        gpgJars(MavenPublication) {
            getSignatureFiles().each {signature ->
                artifact (signature.archive) {
                    classifier = signature.classifier
                    extension  = signature.extension
                }
            }
        }
        if (project.ext.pomFile.exists()) {
            gpgPom(MavenPublication) {
                artifact (getPomSignature()) {
                    classifier = null
                    extension  = 'pom.asc'
                }
            }
        }
        jar(MavenPublication) {
            from components.java
            pom.withXml {
                asNode().children().last() + {
                    resolveStrategy = Closure.DELEGATE_FIRST
                    name 'project-name'
                    description 'description for project'
                    url projectUrl
                    scm {
                        url scmUrl
                        connection connectionUrl
                        developerConnection developerConnectionUrl
                    }
                    licenses {
                        license {
                            name 'The Apache Software License, Version 2.0'
                            url 'http://www.apache.org/license/LICENSE-2.0.txt'
                            distribution 'repo'
                        }
                    }
                    developers {
                        developer {
                            id 'your id or nick name'
                            name 'Your Name'
                            email 'your@mail.address'
                        }
                    }
                }
            }
        }
    }
    repositories {
            if (project.ext.pomFile.exists()) {
                url 'https://oss.sonatype.org/service/local/staging/deploy/maven2/'
                credentials {
                    username = sonatypeUsername
                    password = sonatypePassword
                }
            } else {
                url fileDirectory
            }
    }
}
```



{% render_partial _includes/post/post_footer.html %}

