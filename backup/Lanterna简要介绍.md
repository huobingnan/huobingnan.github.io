TUI是Text User Interface的简写，所谓Text User Interface与Graphic User Interface是类似的，它们都是一种图形用户接口。TUI的出现要比GUI更早，它是基于Terminal终端的一种用户交互接口。

Lanterna是服务于Java的一个TUI框架，它的Github仓库地址为：[Lanterna Github Repository URL](https://github.com/mabe02/lanterna)。截止2024年，Lanterna的作者还在维护它的框架。同Java的跨平台性保持一致，Lanterna也可以在常见的三大操作系统平台（Windows、Mac和Linux）下运行，在不同的平台下，Lanterna会采用不同的策略（后端）来实现TUI。例如在Windows平台下，稳定的Lanterna版本会使用Cygwin终端模拟器来实现，Cygwin是一个在Windows上提供Unix-Like开发环境的工具集合。Lanterna总是倾向于使用Unix环境下的`stty`来实现终端的控制和写入，Cygwin就提供了这样的便利。但是Cygwin并不会被Windows系统默认集成，你通常需要手动在[官网](https://www.cygwin.com/)下载安装它。

在最新的alpha版本的Lanterna中，你可以在Windows下不使用Cygwin就能使用Lanterna。在这个版本中你需要额外进入JNA（Java Native Access）和JNA-Platform，显然这个版本的解决方案采用了Windows的系统API来实现在终端的UI操作。

## 开始使用Lanterna
首先，你需要创建一个Java工程，这里使用的是IDEA开发工具并使用Gradle来构建。
在`build.gradle`引入Lanterna alpha版本的依赖坐标：
```groovy
dependencies {  
    testImplementation platform('org.junit:junit-bom:5.10.0')  
    testImplementation 'org.junit.jupiter:junit-jupiter'  
    // https://mvnrepository.com/artifact/org.projectlombok/lombok  
    compileOnly 'org.projectlombok:lombok:1.18.32'  
    annotationProcessor('org.projectlombok:lombok:1.18.32') 
    // Lanterna alpha版本依赖 
	implementation 'com.googlecode.lanterna:lanterna:3.2.0-alpha1'  
    // 引入JNA和JNA-Platform
    implementation 'net.java.dev.jna:jna:5.14.0'  
    implementation 'net.java.dev.jna:jna-platform:5.14.0'  
}
```

在使用Gradle构建Lanterna TUI应用时，将应用的运行派发给Gradle会造成莫名其妙的错误，因此建议在`build.gradle`中新建一个任务，在应用代码编译完毕后（`compileJava`任务执行完毕之后）生成一个在终端运行Java程序的*bat*脚本。
```groovy
// 生成控制台运行脚本  
tasks.register("genRunBat") {  
    doLast {  
        def file = new File("run.bat")  
        file.exists() && file.delete() && file.createNewFile()  
        def classpath = [  
                "${project.layout.buildDirectory.get().asFile.getAbsolutePath()}\\classes\\java\\main".toString()  
        ]  
        sourceSets.main.runtimeClasspath.collect {  
            if (it.getName().endsWith(".jar")) {  
                classpath.add(it.getAbsolutePath())  
            }  
        }  
        file << "java -cp "  
        file << classpath.join(";") << " "  
        file << "buddha.javapx.tui.MainUI"  
    }  
}
```
