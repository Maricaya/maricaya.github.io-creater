---
title: "java包管理的前生今世--Maven入门"
date: 2019-12-16T14:55:11+08:00
draft: false
---

Maven是Java的包管理工具，类似于JavaScript的npm。Maven已经有十多年的历史，至今仍然被Java程序员广泛使用。要了解Maven，先了解最基本的概念。什么是包？

## 什么是包？
JVM被设计的相当简单：1.执行一个类的字节码 2.假如这个过程中碰到了新的类，加载 -> 1.2...一直循环执行

那么，去哪里加载新的类呢？
类似命令行中执行程序是从环境变量中找到的，新的类是通过类路径（classpath）挨个去找的。
如果碰到jar包（jar包的本质就是zip压缩包，将jar改为zip后解压缩，可以看到一堆java类），就将jar包解压缩后寻找。

包，就是把许多类放在一起，压缩为jar包。
```bash
-classpath/-cp
```

## 什么是包管理？
在我们写的项目中，会使用的一些第三方类，这些第三方类都需要我们来告诉JVM从哪里找到。手动在命令行中写非常麻烦，包管理就是来帮我们解决这个麻烦的过程。

包管理的本质就是，**告诉JVM从哪里找需要的第三方类。以及成功的解决其中的冲突问题。**

## 冲突问题
即你依赖的类还依赖了其他类。程序庞大的情况下，依赖可能非常多。在真实的项目中，可能依赖成百上千个jar包。如果使用命令行的方式一个一个传递，会导致效率十分低下。

这就是使用自动化工具的原因：帮我们完成重复、啰嗦的事情。

我们知道，类的唯一表示就是全限定类名。全限定类名是用来在jar包中，根据目录来定位文件的。如果依赖的两个包，分别依赖同一个同名文件，但是是不同的版本。那应该选择哪个版本呢？

答案很简单：**谁在前面，就用谁。**

在不同的版本中，代码是不同的，因此可能会出现bug。这就是**Classpath hell** 类路径地狱。所以，**绝对不允许最终的classpath出现同名不同版本的jar包。**

对于传递性依赖的自动管理问题，最早的解决方法是：每次上线前都看一下，确定没有同名类。但是Classpath很长，肉眼很难看出问题。Maven的出现，帮我们很好解决了这个问题。

## 在没有包管理工具的时代
java程序员是手动写命令行进行程序的编译、运行。

依赖的所有jar包都需要手动下载，并且手动写上classpath。
```bash
# 编译 需要手动下载commons-lang3-3.9.jar 和lang3依赖的jar包
javac -cp commons-lang3-3.9.jar StringIsBlank.java
# 运行
java -cp commons-lang3-3.9.jar:. StringIsBlank.java 传递参数
```

## Apache Ant时代
手动下载jar包。并写xml配置，指定编译的源代码目录、依赖的jar包、输入目录等。

Ant时代已经节省了很多工作，但也有他的不足。
- 每个人都要造轮子。
  - 因为每个人的目录名称不同，并不通用。
- 依赖的第三方类库都需要手动下载。
- 没有解决classpath hell问题。

## Maven 划时代的包管理
首先要强调，Maven不仅仅是包管理工具。
- Maven为仓库提供了一套约定，可以复用其他人的代码。约定优于配置。
  - 不用手动去下载包，只要配置组织、名称、版本号。Maven会自动下载依赖的包。
- Maven有两个仓库：中央仓库、本地仓库(默认~/.m2) 下载的第三方包，放入m2进行缓存。
- Maven就像一个大书库，根据书的分类去找到下载的第三方包。

### Maven的包
- 按照约定为所有包编号，方便检索
  - groupId
  - artifactId
  - version

### Maven中包冲突解决
- 报出如下的异常时，大部分是包冲突的表现：
  - AbstractMethodError
  - NoClassDefFoundError
  - ClassNotFoundException
  - LinkageError

当包冲突时，Maven会选择哪个版本的包呢？答案和上面的相同：**谁离项目近（在前面），就用谁。**
如果我们想使用后面版本的包，如何修改？
查看Maven中引用结构：
- 在Maven右侧查看树结构，展示的是冲突解决之前的。
- 在shell中输入`mvn dependency:tree`，展示的是冲突解决之后的。

1、离项目近的地方，直接引入需要的版本
2、排除错误的版本
<exclusions>
  <exclusion>
    <groupId>com.github.xxx<groupId>
    <artifactId>test-xxx<artifactId>
    <version>xxx</version>
  </exclusion>
</exclusions>
3、Idea中安装插件 `maven helper`，使用插件解决。

### Maven中的scope，对文件做了一些限定
只能在test中使用
```xml
<scope>test</scope>
```
只在生产环境中使用,上线后由其他人提供
```xml
<scope>provided</scope>
```
编译的时候可运行,运行的时候也在
```xml
<scope>compile</scope>
```