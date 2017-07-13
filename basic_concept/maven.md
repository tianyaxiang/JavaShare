# Apache Maven
## Apache Maven 是做什么用的？
    Maven 是一个项目管理和构建自动化工具。对于我们程序员来说，我们最关心的是它的项目构建功能。
    所以这里我们介绍的就是怎样用 maven 来满足我们项目的日常需要。
Maven 使用惯例优于配置的原则 。它要求在没有定制之前，所有的项目都有如下的结构：
 
|目录|目的|
|---|:---|
|${basedir}|存放 pom.xml和所有的子目录|
|${basedir}/src/main/java|项目的 java源代码|
|${basedir}/src/main/resources|项目的资源，比如说 property文件|
|${basedir}/src/test/java|项目的测试类，比如说 JUnit代码|
|${basedir}/src/test/resources| 测试使用的资源|

一个 maven 项目在默认情况下会产生 JAR 文件，另外 ，编译后 的 classes 会放在 ${basedir}/target/classes 下面， JAR 文件会放在 ${basedir}/target 下面。
    这时有人会说了 ， Ant 就没有那么多要求 ，它允许你可以自由的定义项目的结构。在这里不想引起口水战哈， 我个人觉得 maven 的这些默认定义很方便使用。 
    



参考地址：http://www.oracle.com/technetwork/cn/community/java/apache-maven-getting-started-1-406235-zhs.html
