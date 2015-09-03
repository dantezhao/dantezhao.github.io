---
layout: post
author: zhao
title:  Java�����й��ߣ�Javac
date:   2015-09-03 22:14:54
categories: Java
---

* content
{:toc}

##ǰ��

> ͻȻ��һ�췢�֣�֮ǰ��Ϊѧ����֪ʶ��ʵ����ɽһ�Ƕ��㲻�ϡ�

##���

����ƽ����д��Java������Ҫ�ȱ�����Ϊ�����Ƶ��ֽ��룬����Hello.javaԴ�ļ��ᱻ����ΪHello.class�ֽ����ļ���Ȼ����ܱ�Java�����ִ�С�

�ڴ�֮ǰһ�㶼����Eclipseֱ�ӽ���Java�����ˣ��������õķǳ��٣�������Ҳ������֪��javac�������`.java`�ļ������ڽӴ����˸����Java֪ʶ����ʱ����ҪԶ�̵���Java���򣬻�����Ҫ�鿴`.class`�ļ�������javac��������кܶ���Ҫ��ע��ѧϰ�ĵط���

`javac`��`$JAVA_HOME/bin`�У�һ���ڰ�װ��jdk�������úû���������Ϳ���ֱ��ʹ���ˡ�

##���ò���

������ܼ������û���˵�Լ����õĲ�����

~~~
[root@z1 ~]# javac -help
Usage: javac <options> <source files>

where possible options include:
  -g                         Generate all debugging info	����debugging��Ϣ��-g������������ʹ��javap��������ϸ�Աȳ���
  -g:none                    Generate no debugging info
  -g:{lines,vars,source}     Generate only some debugging info
  -verbose                   Output messages about what the compiler is doing  ��������ʾjavac����������ִ�еĲ�����Ϣ
  -classpath <path>          Specify where to find user class files and annotation processors
  -cp <path>                 Specify where to find user class files and annotation processors
  -d <directory>             Specify where to place generated class files	ָ��.class�ļ����Ŀ¼
  -s <directory>             Specify where to place generated source files
  -encoding <encoding>       Specify character encoding used by source files
  -source <release>          Provide source compatibility with specified release	����ָ��ʹ��ʲô�汾�ı�����������Դ�ļ���
  -target <release>          Generate class files for specific VM version	����ָ������������ֽ����ļ����֧����ʲô�汾��Java����������С�
  -version                   Version information
  -help                      Print a synopsis of standard options
~~~
  
##����

��һ��ʹ�ò����Ƚ϶������

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
  
  
  
  
  
  
  
  
  