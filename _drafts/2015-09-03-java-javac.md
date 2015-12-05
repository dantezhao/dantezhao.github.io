---
layout: post
author: zhao
title:  Java工具：javac
date:   2015-09-03 22:14:54
categories: Java
---

* content
{:toc}

##前言

> 突然有一天发现，之前认为学到的知识其实连冰山一角都算不上。

##简介

我们平常编写的Java代码需要先被编译为二进制的字节码，例如Hello.java源文件会被编译为Hello.class字节码文件，然后才能被Java虚拟机执行。

在此之前一般都是用Eclipse直接进行Java开发了，命令行用的非常少，基本上也就是在知道javac后面跟上`.java`文件。现在接触到了更多的Java知识，有时候需要远程调试Java程序，或者需要查看`.class`文件，发现javac这个工具有很多需要关注和学习的地方。

`javac`在`$JAVA_HOME/bin`中，一般在安装好jdk，并配置好环境变量后就可以直接使用了。

##说明

有两种方法可将源代码文件名传递给 javac

如果源文件数量少，在命令行上列出文件名即可。

如果源文件数量多，则将源文件名列在一个文件中，名称间用空格或回车行来进行分隔。然后在 javac 命令行中使用该列表文件名，文件名前冠以 @ 字符。

源代码文件名称必须含有 .java 后缀，类文件名称必须含有 .class 后缀，源文件和类文件都必须有识别该类的根名。例如，名为 MyClass 的类将写在名为 HelloWorld.java的源文件中，并被编译为字节码类文件 HelloWorld.class。

内部类定义产生附加的类文件。这些类文件的名称将内部类和外部类的名称结合在一起，例如 HelloWorld$InnerHelloworld.class。

应当将源文件安排在反映其包树结构的目录树中。例如，如果将所有的源文件放在 /workspace 中，那么 com.zhao.test.HelloWorld 的代码应该在 \workspace\com\zhao\test\HelloWorld.java 中。

缺省情况下，编译器将每个类文件与其源文件放在同一目录中。可用 -d 选项（请参阅后面的选项）指定其它目标目录。

##常用参数

下面介绍几个常用或者说自己会用的参数。

~~~

用法: javac <options> <source files>
其中, 可能的选项包括:
  -g                         生成所有调试信息，-g参数在debug的时候会使用，特别是在使用Eclipse的进行debug的时候，里面的一些选项其实对应的就是-g参数。
  -g:none                    不生成任何调试信息
  -g:{lines,vars,source}     只生成某些调试信息
  -nowarn                    不生成任何警告
  -verbose                   输出有关编译器正在执行的操作的消息
  -classpath <路径>            指定查找用户类文件和注释处理程序的位置
  -cp <路径>                   指定查找用户类文件和注释处理程序的位置
  -d <目录>                    指定放置生成的类文件的位置
  -s <目录>                    指定放置生成的源文件的位置
  -encoding <编码>             指定源文件使用的字符编码
  -source <发行版>              提供与指定发行版的源兼容性
  -target <发行版>              生成特定 VM 版本的类文件
  -version                   版本信息
  -help                      输出标准选项的提要

~~~
  
##例子

###例1

先来个大致参数用法的例子

~~~
[root@z1 i++issue]# javac -source 1.7 -target 1.7 -d ./classdir/ -verbose -encoding UTF-8 -g:vars SumPlusTest.java 
[parsing started RegularFileObject[SumPlusTest.java]]
[parsing completed 13ms]
[search path for source files: .,/mnt/zhao/soft/jdk/lib/dt.jar,/mnt/zhao/soft/jdk/lib/tools.jar]
[search path for class files: /mnt/zhao/soft/jdk/jre/lib/resources.jar,/mnt/zhao/soft/jdk/jre/lib/rt.jar,/mnt/zhao/soft/jdk/jre/lib/sunrsasign.jar,/mnt/zhao/soft/jdk/jre/lib/jsse.jar,/mnt/zhao/soft/jdk/jre/lib/jce.jar,/mnt/zhao/soft/jdk/jre/lib/charsets.jar,/mnt/zhao/soft/jdk/jre/lib/jfr.jar,/mnt/zhao/soft/jdk/jre/classes,/mnt/zhao/soft/jdk/jre/lib/ext/dnsns.jar,/mnt/zhao/soft/jdk/jre/lib/ext/sunpkcs11.jar,/mnt/zhao/soft/jdk/jre/lib/ext/sunec.jar,/mnt/zhao/soft/jdk/jre/lib/ext/localedata.jar,/mnt/zhao/soft/jdk/jre/lib/ext/zipfs.jar,/mnt/zhao/soft/jdk/jre/lib/ext/sunjce_provider.jar,.,/mnt/zhao/soft/jdk/lib/dt.jar,/mnt/zhao/soft/jdk/lib/tools.jar]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/lang/Object.class)]]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/lang/String.class)]]
[checking SumPlusTest]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/lang/AutoCloseable.class)]]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/lang/System.class)]]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/io/PrintStream.class)]]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/io/FilterOutputStream.class)]]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/io/OutputStream.class)]]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/lang/StringBuilder.class)]]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/lang/CharSequence.class)]]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/io/Serializable.class)]]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/lang/Comparable.class)]]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/lang/AbstractStringBuilder.class)]]
[loading ZipFileIndexFileObject[/mnt/zhao/soft/jdk/lib/ct.sym(META-INF/sym/rt.jar/java/lang/StringBuffer.class)]]
[wrote RegularFileObject[./classdir/SumPlusTest.class]]
[total 639ms]
~~~  

###-d参数

设置类文件的目标目录。如果某个类是一个包的组成部分，则 javac 将把该类文件放入反映包名的子目录中，必要时创建目录。例如，如果指定 `-d ./HelloWorldDir` 并且该类名叫 `com.zhao.test`，那么类文件就叫作 `./HelloWorldDir/package com.zhao.test.HelloWorld.class`。

若未指定 -d 选项，则 javac 将把类文件放到与源文件相同的目录中。

注意： -d 选项指定的目录不会被自动添加到用户类路径中。

~~~
[root@z1 java]# cat HelloWorld.java 
package com.zhao.test;

public class HelloWorld {
	public static void main(String[] args){
		System.out.println("Helloworld!");
	}
}

[root@z1 java]# javac -d HelloWorldDir/ HelloWorld.java 

[root@z1 java]# tree HelloWorldDir/
HelloWorldDir/
└── com
    └── zhao
        └── test
            └── HelloWorld.class

3 directories, 1 file
[root@z1 java]# 
~~~