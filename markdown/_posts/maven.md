---
title:
  Maven入门
date:
  2015/7/12 11:09
categories: Maven
tags: [Maven,Java]
---
![image](http://ww1.sinaimg.cn/mw690/7c2c72d3gw1f2ojrxkq99j209g02eaa4.jpg)
# Maven -- 基于项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具
Maven下载地址：[Maven](http://maven.apache.org/download.cgi )

## Maven配置环境变量
1. 在环境变量-系统变量中添加 MAVEN_HOME 变量，值为 Maven 解压之后的路径
2. 在 Path 变量中添加值 %MAVEN_HOME%\bin;
3. 打开 cmd ，输入 mvn -v 指令，得到：
```bash
>mvn -v
Apache Maven 3.3.3 (7994120775791599e205a5524ec3e0dfe41d4a06; 2015-04-22T19:57:37+08:00)
Maven home: F:\lib\apache-maven-3.3.3\bin\..
Java version: 1.8.0_20, vendor: Oracle Corporation
Java home: C:\Program Files\Java\jdk1.8.0_20\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 7", version: "6.1", arch: "amd64", family: "dos"
```
如果看到类似上面的输出的话，说明 Maven 配置完成。
<!-- more -->

## Maven 项目目录结构
Maven 使用惯例优于配置的原则 。它要求在没有定制之前，所有的项目都有如下的结构：

| 目录                          | 内容                                 |
| ----------------------------- | ------------------------------------ |
| ${basedir}                    | 存放 pom.xml和所有的子目录           |
| ${basedir}/src/main/java      | 项目的 java 源代码                   |
| ${basedir}/src/main/resources | 项目的资源，比如说 property 文件     |
| ${basedir}/src/test/java      | 项目的测试类，比如说 JUnit 代码      |
| ${basedir}/src/test/resources | 测试使用的资源                       |

## 第一个 Maven 项目 -- HelloWorld
创建一个名叫 helloworld 的 Maven 项目 demo
### 分步创建 Maven 项目
* 创建 Maven 原型
 打开 cmd，进入用于存放项目的目录下，输入如下指令：
```bash
>mvn archetype:generate
```
 第一次建立 Maven 项目的时候，maven会自动下载一些依赖包到本地仓库中，等待其下载完毕。
 Maven 默认的本地库是 ~/.m2/repository/ ，在 Windows下是 %USER_HOME%\.m2\repository\ 
* 确认 Maven 项目的模型
 Maven 为我们提供了很多种的项目模型，其中包括了从简单的 Swing 到复杂的 Web 应用。本次项目选择默认模型maven-archetype-quickstart
```bash
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 625:
```
 运行至上面的命令时，回车，选择默认模型。
* 选择 Maven 项目模型版本
```bash
Choose org.apache.maven.archetypes:maven-archetype-quickstart version:
1: 1.0-alpha-1
2: 1.0-alpha-2
3: 1.0-alpha-3
4: 1.0-alpha-4
5: 1.0
6: 1.1
Choose a number: 6:
```
 选择 maven-archetype-quickstart 1.1版本，回车
* 依次设置'groupId','artifactId','version','package'

```bash
Define value for property 'groupId': : net.saismon.helloworld
Define value for property 'artifactId': : helloworld
Define value for property 'version':  1.0-SNAPSHOT: :
Define value for property 'package':  net.saismon.helloworld: :
Confirm properties configuration:
groupId: net.saismon.helloworld
artifactId: helloworld
version: 1.0-SNAPSHOT
package: net.saismon.helloworld
 Y: :
```
 回车，完成项目的创建。这时，在项目存放目录下，就会出现 Maven 为我们创建的 helloworld 项目骨架了。

### 一步创建 Maven 项目
在指令后面跟上属性，这些属性是在命令行中用 -D 选项指定的，该选项使用 -Dname=value 的格式，直接通过一行指令完成 Maven 项目的创建。
```bash
mvn archetype:generate -DgroupId=net.saisimon.helloworld -DartifactId=helloworld -Dpackage=net.saisimon.helloworld -Dversion=1.0-SNAPSHOT
```

## helloworld 项目的目录结构
目录结构如下：
```bash
helloworld
    ├ pom.xml
    └ src
       ├ main
       │  └ java\net\saisimon\helloworld\App.java
       └ test
	   └ java\net\saisimon\helloworld\AppTest.java
```
pom.xml --  Project Object Model(POM)文件，用于描述项目，配置插件和管理依赖关系
src/main -- 用于存放源码和源码使用的资源文件
src/test -- 用于存放测试代码和测试使用的资源文件

## 对 helloworld 项目进行打包
将 helloworld 项目进行打包，在 cmd 窗口下，输入：
```bash
>cd helloworld
>mvn package
```
maven 在 helloworld 下面建立了一个新的目录 target/ ，构建打包后的 jar 文件 helloworld-1.0-SNAPSHOT.jar 就存放在这个目录下
通过 java 命令来运行打包后的程序：
```bash
>java -cp target/helloworld-1.0-SNAPSHOT.jar net.saisimon.helloworld.App
Hello World!
```
运行成功，简单的 Maven 项目的 demo 就完成了

## Maven 中的POM(Project Object Model)
前面也提到了 POM -- 用于描述项目，配置插件和管理依赖关系。下面是 helloworld 项目中的pom.xml文件：
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <!--项目的全球唯一标识符，通常使用全限定的包名区分该项目和其他项目。并且构建时生成的路径也是由此生成， 如com.mycompany.app生成的相对路径为：/com/mycompany/app-->
  <groupId>net.saisimon.helloworld</groupId>
  <!--构件的标识符，它和group ID一起唯一标识一个构件。换句话说，你不能有两个不同的项目拥有同样的artifact ID和groupID；在某个特定的group ID下，artifact ID也必须是唯一的。构件是项目产生的或使用的一个东西，Maven为项目产生的构件包括：JARs，源码，二进制发布和WARs等。-->   
  <artifactId>helloworld</artifactId>
  <!--项目当前版本，格式为:主版本.次版本.增量版本-限定版本号-->    
  <version>1.0-SNAPSHOT</version>
  <!--项目产生的构件类型，例如jar、war、ear、pom。插件可以创建他们自己的构件类型，所以前面列的不是全部构件类型-->   
  <packaging>jar</packaging>
  
  <!--项目的名称, Maven产生的文档用-->
  <name>helloworld</name>
  <!--项目主页的URL, Maven产生的文档用-->
  <url>http://blog.saisimon.net</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  
  <!--该元素描述了项目相关的所有依赖。 这些依赖组成了项目构建过程中的一个个环节。它们自动从项目定义的仓库中下载。-->
  <dependencies>
    <dependency>
      <!--依赖的group ID-->
      <groupId>junit</groupId>
      <!--依赖的artifact ID-->  
      <artifactId>junit</artifactId>
      <!--依赖的版本号。 在Maven 2里, 也可以配置成版本号的范围。--> 
      <version>3.8.1</version>
      <!--
	依赖范围。在项目发布过程中，帮助决定哪些构件被包括进来。   
	- compile: 默认范围，用于编译，任何时候都会被包含在 classpath 中，在打包的时候也会被包括进去   
	- provided: 类似于编译，但支持你期待jdk或者容器提供，类似于classpath     
	- runtime: 在执行时需要使用     
	- test:  用于test任务时使用     
	- system: 需要外在提供相应的元素。通过systemPath来取得     
	- systemPath: 仅用于范围为system。提供相应的路径     
	- optional:  当项目自身被依赖时，标注依赖是否传递。用于连续依赖时使用
      --> 
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```
上面文件中的 groupId, artifactId, packaging, version 叫作 maven 坐标，它能唯一的确定一个项目。
groupId：项目或者组织公司的唯一标志
artifactId：项目名称
version：项目版本号
packaging：项目打包的方式

更多pom.xml的详细介绍请戳：[Maven pom.xml文件详解](http://www.blogjava.net/hellxoul/archive/2013/05/16/399345.html )

## Maven的生命周期
Maven定义了三套生命周期：clean、default、site，每个生命周期都包含了一些阶段（phase）。三套生命周期相互独立，但各个生命周期中的phase却是有顺序的，且后面的phase依赖于前面的phase。执行某个phase时，其前面的phase会依顺序执行，但不会触发另外两套生命周期中的任何phase。
* clean生命周期

| 操作                          | 说明                                 |
| ----------------------------- | ------------------------------------ |
| pre-clean                     | 执行清理前的工作                     |
| clean                         | 清理上一次构建生成的所有文件         |
| post-clean                    | 执行清理后的工作                     |

* default生命周期

| 操作                          | 说明                                     |
| ----------------------------- | ---------------------------------------- |
| validate                      | 验证项目一些必要的信息                   |
| initialize                    | 初始化                                   |
| generate-sources              | 生成编译过程需要的源代码                 |
| process-sources               | 处理源代码                               |
| generate-resources            | 生成打包需要的资源文件                   |
| process-resources             | 复制和处理资源文件到target目录，准备打包 |
| compile                       | 编译项目的源代码                         |
| process-classes               | 后处理字节码文件                         |
| generate-test-sources         |                                          |
| process-test-sources          |                                          |
| generate-test-resources       |                                          |
| process-test-resources        |                                          |
| test-compile                  | 编译测试源代码                           |
| process-test-classes          |                                          |
| test                          | 运行测试代码                             |
| prepare-package               | 打包前准备操作                           |
| package                       | 打包成jar或者war或者其他格式的分发包     |
| pre-integration-test          | 集成测试运行前操作                       |
| integration-test              | 集成测试                                 |
| post-integration-test         | 集成测试后操作                           |
| verify                        | 检查                                     |
| install                       | 将打好的包安装到本地仓库，供其他项目使用 |
| deploy                        | 将打好的包安装到远程仓库，供其他项目使用 |

* site生命周期

| 操作                          | 说明                                 |
| ----------------------------- | ------------------------------------ |
| pre-site                      | 生成项目的站点文档之前               |
| site                          | 生成项目的站点文档；                 |
| post-site                     | 生成项目的站点文档之后               |
| site-deploy                   | 发布生成的站点文档                   |

## Maven 库
Maven 的仓库分为本地仓库和远程仓库，前面已经提到，本地仓库的位置在：~/.m2/repository/ ，在 Windows下是 %USER_HOME%\.m2\repository\，而 Maven 的远程仓库在：[Maven中央仓库](http://search.maven.org/#browse )，当 maven 查找需要的 jar 文件时，它会先在本地库中寻找，只有在找不到的情况下，才会去远程库中找。
可以通过运行一下命令，来将其他项目安装到本地库
```bash
>mvn install
```

## 参考资料
* [Apache Maven Project](http://maven.apache.org/index.html )
* [Apache Maven 入门篇 (上)](http://www.oracle.com/technetwork/cn/community/java/apache-maven-getting-started-1-406235-zhs.html )
* [Apache Maven 入门篇(下)](http://www.oracle.com/technetwork/cn/community/java/apache-maven-getting-started-2-405568-zhs.html )
* [Maven pom.xml文件详解](http://www.blogjava.net/hellxoul/archive/2013/05/16/399345.html )