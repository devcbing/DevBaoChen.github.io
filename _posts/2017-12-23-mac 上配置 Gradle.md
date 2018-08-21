---
layout: post
title: 在mac 上配置 Gradle
categories: [Mac, Gradle]
description: 在mac 上配置 Gradle
keywords: Mac, Gradle
---

在 Mac 上配置 Gradle 相关步骤!


![](/images/posts/tools/2017-12-23-01.png)

### 配置Java环境
 在配置 Gradle 需要先配置 Java 环境，并且 jdk 版本需要 大于 1.6 ，并且 需要在环境变量中配置 JAVA_HOME ,使用下面的命令可以查看 Java 版本信息
 ```
    java -version
 ```
 输出信息如下，说明成功.
 ```
    java version "1.8.0_181"
    Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
    Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
 ```

 Java 环境的相关配置，可以查看 [MAC安装JDK及环境变量配置](https://blog.csdn.net/vvv_110/article/details/72897142);

 ### 下载 Gradle

 下载地址 [点击下载gradle](http://services.gradle.org/distributions/) ; 尽量下载最新的版本， 这需要注意的是，要下载 `all` 的版本，里面包含所有的文件，包含 `源码、文档`等...

 ### 配置 Gradle 
 
 下载好 Gradle 后解压，放到指定的目录：然后打开 `bash_profile` 文件进行环境变量配置 
可以参考：
 [mac os 中操作bash_profile步骤](https://devbaochen.github.io/2017/01/18/mac-os-%E4%B8%AD%E6%93%8D%E4%BD%9Cbash_profile%E6%AD%A5%E9%AA%A4/)

 在 `bash_profile` 中 配置如下代码

 ```
    GRADLE_HOME=你的Gradle路经
    export GRADLE_HOME
    export PATH=$PATH:$GRADLE_HOME/bin
 ```

 保存并退出，然后使用 `source .bash_profile` 更新你的环境变量

 ### 检验 Gradle 是否配置成功
 打开终端,运行命令 `gradle -version` ，如果出现下面内容，说明配置成功, 

 ```
    ------------------------------------------------------------
    Gradle 4.4.1
    ------------------------------------------------------------

    Build time:   2017-12-20 15:45:23 UTC
    Revision:     10ed9dc355dc39f6307cc98fbd8cea314bdd381c

    Groovy:       2.4.12
    Ant:          Apache Ant(TM) version 1.9.9 compiled on February 2 2017
    JVM:          1.8.0_181 (Oracle Corporation 25.181-b13)
    OS:           Mac OS X 10.13.6 x86_64

 ```







