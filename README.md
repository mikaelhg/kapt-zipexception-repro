# `kapt` fails with `gradle build` when Graal SVM JAR in compile classpath

JetBrains issues:

* [KT-41313](https://youtrack.jetbrains.com/issue/KT-41313) (Kotlin 1.4.0)

* [KT-29513](https://youtrack.jetbrains.com/issue/KT-29513) (Kotlin 1.3.20)

Reproduction: https://github.com/mikaelhg/kapt-zipexception-repro

    plugins {
        id 'org.jetbrains.kotlin.jvm' version '1.4.0'
        id 'org.jetbrains.kotlin.kapt' version '1.4.0'
    }
               ...
    dependencies {
        implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8"
        kapt "io.micronaut:micronaut-inject-java:2.0.1"
        kapt "io.micronaut:micronaut-graal:2.0.1"
        compileOnly "org.graalvm.nativeimage:svm:20.2.0"
    }

## How to reproduce

    $ ./gradlew build -S

### Java 14

    $ JAVA_HOME=~/.jdks/adopt-openjdk-14 ./gradlew clean build -S
    
    Caused by: java.util.zip.ZipException: zip END header not found
            at java.base/java.util.zip.ZipFile$Source.zerror(ZipFile.java:1482)
            at java.base/java.util.zip.ZipFile$Source.findEND(ZipFile.java:1383)
            at java.base/java.util.zip.ZipFile$Source.initCEN(ZipFile.java:1390)
            at java.base/java.util.zip.ZipFile$Source.<init>(ZipFile.java:1221)
            at java.base/java.util.zip.ZipFile$Source.get(ZipFile.java:1184)
            at java.base/java.util.zip.ZipFile$CleanableResource.<init>(ZipFile.java:718)
            at java.base/java.util.zip.ZipFile.<init>(ZipFile.java:238)
            at java.base/java.util.zip.ZipFile.<init>(ZipFile.java:168)
            at java.base/java.util.zip.ZipFile.<init>(ZipFile.java:182)
            at org.jetbrains.kotlin.kapt3.base.ProcessorLoader.doLoadProcessors(ProcessorLoader.kt:104)
            at org.jetbrains.kotlin.kapt3.base.ProcessorLoader.loadProcessors(ProcessorLoader.kt:43)
            at org.jetbrains.kotlin.kapt3.base.Kapt.kapt(Kapt.kt:42)
            ... 35 more

    $ JAVA_HOME=~/.jdks/adopt-openjdk-14 ./gradlew -v
    
    ------------------------------------------------------------
    Gradle 6.6
    ------------------------------------------------------------
    
    Build time:   2020-08-10 22:06:19 UTC
    Revision:     d119144684a0c301aea027b79857815659e431b9
    
    Kotlin:       1.3.72
    Groovy:       2.5.12
    Ant:          Apache Ant(TM) version 1.10.8 compiled on May 10 2020
    JVM:          14 (AdoptOpenJDK 14+36)
    OS:           Linux 5.8.0-050800-generic amd64

### Java 11

    $ JAVA_HOME=~/.jdks/adopt-openjdk-11.0.6 ./gradlew clean build -S

    Caused by: java.util.zip.ZipException: zip END header not found
            at java.base/java.util.zip.ZipFile$Source.zerror(ZipFile.java:1535)
            at java.base/java.util.zip.ZipFile$Source.findEND(ZipFile.java:1436)
            at java.base/java.util.zip.ZipFile$Source.initCEN(ZipFile.java:1443)
            at java.base/java.util.zip.ZipFile$Source.<init>(ZipFile.java:1274)
            at java.base/java.util.zip.ZipFile$Source.get(ZipFile.java:1237)
            at java.base/java.util.zip.ZipFile$CleanableResource.<init>(ZipFile.java:727)
            at java.base/java.util.zip.ZipFile$CleanableResource.get(ZipFile.java:844)
            at java.base/java.util.zip.ZipFile.<init>(ZipFile.java:247)
            at java.base/java.util.zip.ZipFile.<init>(ZipFile.java:177)
            at java.base/java.util.zip.ZipFile.<init>(ZipFile.java:191)
            at org.jetbrains.kotlin.kapt3.base.ProcessorLoader.doLoadProcessors(ProcessorLoader.kt:104)
            at org.jetbrains.kotlin.kapt3.base.ProcessorLoader.loadProcessors(ProcessorLoader.kt:43)
            at org.jetbrains.kotlin.kapt3.base.Kapt.kapt(Kapt.kt:42)
            ... 35 more

    $ JAVA_HOME=~/.jdks/adopt-openjdk-11.0.6 ./gradlew -v
    
    ------------------------------------------------------------
    Gradle 6.6
    ------------------------------------------------------------
    
    Build time:   2020-08-10 22:06:19 UTC
    Revision:     d119144684a0c301aea027b79857815659e431b9
    
    Kotlin:       1.3.72
    Groovy:       2.5.12
    Ant:          Apache Ant(TM) version 1.10.8 compiled on May 10 2020
    JVM:          11.0.6 (AdoptOpenJDK 11.0.6+10)
    OS:           Linux 5.8.0-050800-generic amd64

### Java 8

    $ JAVA_HOME=~/.jdks/adopt-openjdk-1.8.0_265 ./gradlew clean build -S

    Caused by: java.util.zip.ZipException: error in opening zip file
            at java.util.zip.ZipFile.open(Native Method)
            at java.util.zip.ZipFile.<init>(ZipFile.java:225)
            at java.util.zip.ZipFile.<init>(ZipFile.java:155)
            at java.util.zip.ZipFile.<init>(ZipFile.java:169)
            at org.jetbrains.kotlin.kapt3.base.ProcessorLoader.doLoadProcessors(ProcessorLoader.kt:104)
            at org.jetbrains.kotlin.kapt3.base.ProcessorLoader.loadProcessors(ProcessorLoader.kt:43)
            at org.jetbrains.kotlin.kapt3.base.Kapt.kapt(Kapt.kt:42)
            ... 35 more

    $ JAVA_HOME=~/.jdks/adopt-openjdk-1.8.0_265 ./gradlew -v
    
    ------------------------------------------------------------
    Gradle 6.6
    ------------------------------------------------------------
    
    Build time:   2020-08-10 22:06:19 UTC
    Revision:     d119144684a0c301aea027b79857815659e431b9
    
    Kotlin:       1.3.72
    Groovy:       2.5.12
    Ant:          Apache Ant(TM) version 1.10.8 compiled on May 10 2020
    JVM:          1.8.0_265 (AdoptOpenJDK 25.265-b01)
    OS:           Linux 5.8.0-050800-generic amd64
