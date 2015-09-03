---
layout: post
author: zhao
title:  Java命令行工具：javac
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

##常用参数

下面介绍几个常用或者说自己会用的参数。

~~~
[root@z1 ~]# javac -help
Usage: javac <options> <source files>

where possible options include:
  -g                         Generate all debugging info	生成debugging信息，-g参数的区别，在使用javap参数能详细对比出来
  -g:none                    Generate no debugging info
  -g:{lines,vars,source}     Generate only some debugging info
  -verbose                   Output messages about what the compiler is doing  可用于显示javac编译器正在执行的操作信息
  -classpath <path>          Specify where to find user class files and annotation processors
  -cp <path>                 Specify where to find user class files and annotation processors
  -d <directory>             Specify where to place generated class files	指定.class文件输出目录
  -s <directory>             Specify where to place generated source files
  -encoding <encoding>       Specify character encoding used by source files
  -source <release>          Provide source compatibility with specified release	用于指定使用什么版本的编译器来编译源文件。
  -target <release>          Generate class files for specific VM version	用于指定编译出来的字节码文件最低支持在什么版本的Java虚拟机上运行。
  -version                   Version information
  -help                      Print a synopsis of standard options
~~~
  
##例子

举一个使用参数比较多的例子

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