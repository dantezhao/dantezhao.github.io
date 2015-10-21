---
layout: post
author: zhao
title:  Java工具：javap
date:   2015-09-03 22:14:54
categories: Java
---

* content
{:toc}


##简介

javap，是JDK自带的反汇编工具，用于将Java字节码文件反汇编为Java源代码。

具体到我所能用的功能来说，就是Java编译器为我们生成的字节码。特别是在针对一些比较细节的问题，比如java的中间缓存变量机制或者String的一些问题时，通过javap来来分析，会直观的多

##常用参数

下面介绍几个常用或者说自己会用的参数。

~~~
用法: javap <options> <classes>
可能的选项包括:
  -help  --help  -?        打印用法信息
  -version                 版本信息
  -v  -verbose             打印附加信息
  -l                       打印行号和本地变量表
  -public                  仅显示public类和成员
  -protected               显示protected/public类和成员
  -package                 显示package/protected/public类和成员(默认值)
  -p  -private             显示所有的类和成员
  -c                       反汇编代码
  -s                       打印内部类型签名
  -sysinfo                 显示将被处理的类的系统信息(路径，大小，日期，MD5 哈希值)
  -constants               显示static final常量
  -classpath <path>        指定查找用户类文件的位置
  -bootclasspath <path>    覆盖由引导类加载器所加载的类文件的位置
~~~
  
##例子

###无参数

没什么太多的信息

~~~
[root@z1 classdir]# javap SumPlusTest 
public class SumPlusTest {
  public SumPlusTest();
  public static void main(java.lang.String[]);
}
~~~

###-c参数

反汇编字节码文件为JVM可以识别、执行的字节码命令

~~~
[root@z1 classdir]# javap -c SumPlusTest
public class SumPlusTest {
  public SumPlusTest();
    Code:
       0: aload_0       
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return        

  public static void main(java.lang.String[]);
    Code:
       0: iconst_1      
       1: istore_1      
       2: iconst_1      
       3: istore_2      
       4: iconst_1      
       5: istore_3      
       6: iconst_1      
       7: istore        4
       9: iinc          1, 1
      12: iinc          2, 1
      15: iload_3       
      16: iinc          3, 1
      19: istore_3      
      20: iinc          4, 1
      23: iload         4
      25: istore        4
      27: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      30: new           #3                  // class java/lang/StringBuilder
      33: dup           
      34: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
      37: ldc           #5                  // String a : 
      39: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      42: iload_1       
      43: invokevirtual #7                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      46: ldc           #8                  // String \nb : 
      48: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      51: iload_2       
      52: invokevirtual #7                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      55: ldc           #9                  // String \nc : 
      57: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      60: iload_3       
      61: invokevirtual #7                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      64: ldc           #10                 // String \nd : 
      66: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      69: iload         4
      71: invokevirtual #7                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      74: invokevirtual #11                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      77: invokevirtual #12                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      80: return        
}
~~~

###-verbose参数

-verbose打印方法参数、常量池、栈区等各种信息。

具体内容，放在i++相关的中间缓存变量机制中说明。

~~~
[root@z1 classdir]# javap -verbose SumPlusTest
Classfile /mnt/workspace/java/i++issue/classdir/SumPlusTest.class
  Last modified Sep 3, 2015; size 794 bytes
  MD5 checksum 3a406dc81fe69722d53ce74ef96516a4
public class SumPlusTest
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #14.#30        //  java/lang/Object."<init>":()V
   #2 = Fieldref           #31.#32        //  java/lang/System.out:Ljava/io/PrintStream;
   #3 = Class              #33            //  java/lang/StringBuilder
   #4 = Methodref          #3.#30         //  java/lang/StringBuilder."<init>":()V
   #5 = String             #34            //  a : 
   #6 = Methodref          #3.#35         //  java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #7 = Methodref          #3.#36         //  java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
   #8 = String             #37            //  \nb : 
   #9 = String             #38            //  \nc : 
  #10 = String             #39            //  \nd : 
  #11 = Methodref          #3.#40         //  java/lang/StringBuilder.toString:()Ljava/lang/String;
  #12 = Methodref          #41.#42        //  java/io/PrintStream.println:(Ljava/lang/String;)V
  #13 = Class              #43            //  SumPlusTest
  #14 = Class              #44            //  java/lang/Object
  #15 = Utf8               <init>
  #16 = Utf8               ()V
  #17 = Utf8               Code
  #18 = Utf8               LocalVariableTable
  #19 = Utf8               this
  #20 = Utf8               LSumPlusTest;
  #21 = Utf8               main
  #22 = Utf8               ([Ljava/lang/String;)V
  #23 = Utf8               args
  #24 = Utf8               [Ljava/lang/String;
  #25 = Utf8               a
  #26 = Utf8               I
  #27 = Utf8               b
  #28 = Utf8               c
  #29 = Utf8               d
  #30 = NameAndType        #15:#16        //  "<init>":()V
  #31 = Class              #45            //  java/lang/System
  #32 = NameAndType        #46:#47        //  out:Ljava/io/PrintStream;
  #33 = Utf8               java/lang/StringBuilder
  #34 = Utf8               a : 
  #35 = NameAndType        #48:#49        //  append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #36 = NameAndType        #48:#50        //  append:(I)Ljava/lang/StringBuilder;
  #37 = Utf8               \nb : 
  #38 = Utf8               \nc : 
  #39 = Utf8               \nd : 
  #40 = NameAndType        #51:#52        //  toString:()Ljava/lang/String;
  #41 = Class              #53            //  java/io/PrintStream
  #42 = NameAndType        #54:#55        //  println:(Ljava/lang/String;)V
  #43 = Utf8               SumPlusTest
  #44 = Utf8               java/lang/Object
  #45 = Utf8               java/lang/System
  #46 = Utf8               out
  #47 = Utf8               Ljava/io/PrintStream;
  #48 = Utf8               append
  #49 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #50 = Utf8               (I)Ljava/lang/StringBuilder;
  #51 = Utf8               toString
  #52 = Utf8               ()Ljava/lang/String;
  #53 = Utf8               java/io/PrintStream
  #54 = Utf8               println
  #55 = Utf8               (Ljava/lang/String;)V
{
  public SumPlusTest();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0       5     0  this   LSumPlusTest;

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=5, args_size=1
         0: iconst_1      
         1: istore_1      
         2: iconst_1      
         3: istore_2      
         4: iconst_1      
         5: istore_3      
         6: iconst_1      
         7: istore        4
         9: iinc          1, 1
        12: iinc          2, 1
        15: iload_3       
        16: iinc          3, 1
        19: istore_3      
        20: iinc          4, 1
        23: iload         4
        25: istore        4
        27: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        30: new           #3                  // class java/lang/StringBuilder
        33: dup           
        34: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
        37: ldc           #5                  // String a : 
        39: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        42: iload_1       
        43: invokevirtual #7                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        46: ldc           #8                  // String \nb : 
        48: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        51: iload_2       
        52: invokevirtual #7                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        55: ldc           #9                  // String \nc : 
        57: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        60: iload_3       
        61: invokevirtual #7                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        64: ldc           #10                 // String \nd : 
        66: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        69: iload         4
        71: invokevirtual #7                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        74: invokevirtual #11                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        77: invokevirtual #12                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        80: return        
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0      81     0  args   [Ljava/lang/String;
               2      79     1     a   I
               4      77     2     b   I
               6      75     3     c   I
               9      72     4     d   I
}
~~~
