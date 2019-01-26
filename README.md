# [KT-29513](https://youtrack.jetbrains.com/issue/KT-29513)

Reproduction: https://github.com/mikaelhg/kapt-zipexception-repro

The ZipException details are different on Java 11 vs. Java 8.

    plugins {
        id 'org.jetbrains.kotlin.jvm' version '1.3.20'
        id 'org.jetbrains.kotlin.kapt' version '1.3.20'
    }
               ...
    dependencies {
        implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8"
        kapt "io.micronaut:micronaut-inject-java:1.0.3"
        compileOnly('com.oracle.substratevm:svm:1.0.0-rc11') {
            because "For Graal native-image building."
            exclude group: "com.oracle.substratevm", module: "svm-hosted-native-windows-amd64"
        }
    }

## How to reproduce

    $ ./gradlew build

### Java 11

    > Task :kaptKotlin FAILED
    e: java.util.zip.ZipException: zip END header not found
            at java.base/java.util.zip.ZipFile$Source.zerror(ZipFile.java:1529)
            at java.base/java.util.zip.ZipFile$Source.findEND(ZipFile.java:1430)
            at java.base/java.util.zip.ZipFile$Source.initCEN(ZipFile.java:1437)

### Java 8

    > Task :kaptKotlin FAILED
    e: java.util.zip.ZipException: error in opening zip file
            at java.util.zip.ZipFile.open(Native Method)
            at java.util.zip.ZipFile.<init>(ZipFile.java:225)
            at java.util.zip.ZipFile.<init>(ZipFile.java:155)
            at java.util.zip.ZipFile.<init>(ZipFile.java:169)
            at org.jetbrains.kotlin.cli.jvm.plugins.ServiceLoaderLite.findImplementationsInJar(ServiceLoaderLite.kt:90)
            at org.jetbrains.kotlin.cli.jvm.plugins.ServiceLoaderLite.findImplementations(ServiceLoaderLite.kt:74)
            at org.jetbrains.kotlin.cli.jvm.plugins.ServiceLoaderLite.findImplementations(ServiceLoaderLite.kt:66)
            at org.jetbrains.kotlin.cli.jvm.plugins.ServiceLoaderLite.loadImplementations(ServiceLoaderLite.kt:49)
            at org.jetbrains.kotlin.kapt3.ClasspathBasedKapt3Extension$loadProcessors$efficientProcessorLoader$1.doLoadProcessors(Kapt3Extension.kt:83)
            at org.jetbrains.kotlin.kapt3.base.ProcessorLoader.loadProcessors(ProcessorLoader.kt:42)
            at org.jetbrains.kotlin.kapt3.base.ProcessorLoader.loadProcessors$default(ProcessorLoader.kt:25)
            at org.jetbrains.kotlin.kapt3.ClasspathBasedKapt3Extension.loadProcessors(Kapt3Extension.kt:88)
            at org.jetbrains.kotlin.kapt3.AbstractKapt3Extension.analysisCompleted(Kapt3Extension.kt:171)
            at org.jetbrains.kotlin.kapt3.ClasspathBasedKapt3Extension.analysisCompleted(Kapt3Extension.kt:98)
            at org.jetbrains.kotlin.cli.jvm.compiler.TopDownAnalyzerFacadeForJVM$analyzeFilesWithJavaIntegration$2.invoke(TopDownAnalyzerFacadeForJVM.kt:96)
            at org.jetbrains.kotlin.cli.jvm.compiler.TopDownAnalyzerFacadeForJVM.analyzeFilesWithJavaIntegration(TopDownAnalyzerFacadeForJVM.kt:106)
            at org.jetbrains.kotlin.cli.jvm.compiler.TopDownAnalyzerFacadeForJVM.analyzeFilesWithJavaIntegration$default(TopDownAnalyzerFacadeForJVM.kt:82)
            at org.jetbrains.kotlin.cli.jvm.compiler.KotlinToJVMBytecodeCompiler$analyze$1.invoke(KotlinToJVMBytecodeCompiler.kt:384)
            at org.jetbrains.kotlin.cli.jvm.compiler.KotlinToJVMBytecodeCompiler$analyze$1.invoke(KotlinToJVMBytecodeCompiler.kt:70)
            at org.jetbrains.kotlin.cli.common.messages.AnalyzerWithCompilerReport.analyzeAndReport(AnalyzerWithCompilerReport.kt:107)
            at org.jetbrains.kotlin.cli.jvm.compiler.KotlinToJVMBytecodeCompiler.analyze(KotlinToJVMBytecodeCompiler.kt:375)
            at org.jetbrains.kotlin.cli.jvm.compiler.KotlinToJVMBytecodeCompiler.compileModules$cli(KotlinToJVMBytecodeCompiler.kt:123)
            at org.jetbrains.kotlin.cli.jvm.K2JVMCompiler.doExecute(K2JVMCompiler.kt:159)
            at org.jetbrains.kotlin.cli.jvm.K2JVMCompiler.doExecute(K2JVMCompiler.kt:57)
            at org.jetbrains.kotlin.cli.common.CLICompiler.execImpl(CLICompiler.java:96)
            at org.jetbrains.kotlin.cli.common.CLICompiler.execImpl(CLICompiler.java:52)
            at org.jetbrains.kotlin.cli.common.CLITool.exec(CLITool.kt:93)
            at org.jetbrains.kotlin.daemon.CompileServiceImpl$compile$1$1$2.invoke(CompileServiceImpl.kt:442)
            at org.jetbrains.kotlin.daemon.CompileServiceImpl$compile$1$1$2.invoke(CompileServiceImpl.kt:102)
            at org.jetbrains.kotlin.daemon.CompileServiceImpl$doCompile$$inlined$ifAlive$lambda$2.invoke(CompileServiceImpl.kt:1013)
            at org.jetbrains.kotlin.daemon.CompileServiceImpl$doCompile$$inlined$ifAlive$lambda$2.invoke(CompileServiceImpl.kt:102)
            at org.jetbrains.kotlin.daemon.common.DummyProfiler.withMeasure(PerfUtils.kt:137)
            at org.jetbrains.kotlin.daemon.CompileServiceImpl.checkedCompile(CompileServiceImpl.kt:1055)
            at org.jetbrains.kotlin.daemon.CompileServiceImpl.doCompile(CompileServiceImpl.kt:1012)
            at org.jetbrains.kotlin.daemon.CompileServiceImpl.compile(CompileServiceImpl.kt:441)
            at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
            at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
            at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
            at java.lang.reflect.Method.invoke(Method.java:498)
            at sun.rmi.server.UnicastServerRef.dispatch(UnicastServerRef.java:357)
            at sun.rmi.transport.Transport$1.run(Transport.java:200)
            at sun.rmi.transport.Transport$1.run(Transport.java:197)
            at java.security.AccessController.doPrivileged(Native Method)
            at sun.rmi.transport.Transport.serviceCall(Transport.java:196)
            at sun.rmi.transport.tcp.TCPTransport.handleMessages(TCPTransport.java:573)
            at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run0(TCPTransport.java:834)
            at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.lambda$run$0(TCPTransport.java:688)
            at java.security.AccessController.doPrivileged(Native Method)
            at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(TCPTransport.java:687)
            at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
            at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
            at java.lang.Thread.run(Thread.java:748)
    
    
    FAILURE: Build failed with an exception.