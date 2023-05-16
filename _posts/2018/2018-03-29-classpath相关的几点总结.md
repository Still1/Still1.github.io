---
title: classpath相关的几点总结
tags: [Java, JVM]
---

## 知识总结

1. JVM运行的classpath根据系统环境变量CLASSPATH来确定。一般来说JDK相关的基础jar（例如dt.jar/tools.jar）可以省略，JVM会找

2. classpath中如果是文件夹，只会扫描文件夹下的class文件，不会扫描jar文件，因此jar文件需要单独列出

3. 对于war包，部署到中间件后，JVM的classpath范围将会包含war包里面的相关路径，具体是
   * `*.war/WEB-INF/classes`
   * `*.war/WEB-INF/lib/`路径下各jar包
4. 对于maven项目，`src/main/java/`目录下的java文件将会编译到`*.war/WEB-INF/classes`中。`src/main/resource/`目录下的文件将会在编译阶段复制到`*.war/WEB-INF/classes`中。因此可把这两个工程目录看作是JVM运行时的classpath的根目录

