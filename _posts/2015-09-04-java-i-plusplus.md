---
layout: post
author: zhao
title:  Java：i++问题
date:   2015-09-03 22:14:54
categories: Java
---

* content
{:toc}


##i++问题引发的知识链

>一个表面上简单的i++问题，让我从此带着敬畏之心去学习！

###例子
~~~java

public class SumPlusTest {
	
	public static void main(String[] args){
		int a = 1, b = 1, c = 1, d =1;
		a++;
		++b;
		c = c++;
		d = ++d;
		System.out.println("a : " + a + "\nb : " + b + "\nc : " + c + "\nd : " + d);
	}
}

~~~

###输出结果

~~~
a : 2
b : 2
c : 1
d : 2
~~~


###分析

a、b、d三个参数的结果都没什么问题，暂且放一边，主要问题出来c上，为什么结果是1，而不是2？

有两种解释方法，其实最后本质是一样的。

####解法一

因为Java用了中间缓存变量机制，所有c=c++可换成如下写法：

~~~
tmp = c;
c = c++;
c = tmp;
~~~

看着其实比较明确了，但是说实话其实没看懂，什么是中间缓存变量机制，为什么要有这个，官方文档在哪？

这些问题不是很明了，国外的权威资料暂时也没得求证，所以索性从字节码的角度来理解这个问题。


###解法二

通过`javap`工具查看虚指令，通过虚指令理解c=c++到底做了什么。

下面结果只保留部分主要内容来说明`c=c++`和其它几种情况的不同，直接翻译虚指令对应程序中的具体操作。

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
  
  ......
  
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
         1: istore_1      #0,1相当于是执行a=1
         2: iconst_1      
         3: istore_2      #2,3相当于是执行b=1
         4: iconst_1      
         5: istore_3      #4,5相当于是执行c=1
         6: iconst_1      
         7: istore_4	  #6,7相当于是执行d=1
         9: iinc          1, 1		#相当于执行a的值+1
        12: iinc          2, 1		#相当于执行b的值+1
        15: iload_3       			#将c的值放入栈顶
        16: iinc          3, 1		#执行c的值+1
        19: istore_3      			#再将栈顶的值赋值给c，此时即c=1
        20: iinc          4, 1		#相当于是执行d的值+1
        23: iload         4			#把d的值放置于栈顶
        25: istore        4			#然后将栈顶的值付给d这个变量，此时d=2
        
		......
      
	  LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0      81     0  args   [Ljava/lang/String;
               2      79     1     a   I
               4      77     2     b   I
               6      75     3     c   I
               9      72     4     d   I
}

~~~

##总结

通过上面分析可以看出来，一个比较简单的i++问题，如果稍微深入一点学习的话，会涉及到不少平常不注意的内容。

对于a++和++b来说，在虚指令里面都是通过`iinc   n, 1`来进行加一操作。

对于`c = c++`和`d = ++d`来说，他们都进行了赋值操作，结果和上面不进行赋值操作的运算对比的话会出现一些特殊情况。

`c = c++`是先将第3个slot的值放入了栈顶，然后再对3个slot进行加一操作，最后再把栈顶的值写入到第3个slot，所以c的值没有改变。

`d = ++d`但是这种情况就是先将第四个slot的加一再放入栈顶，所有最后的值增加的。

##疑问

那么就有了一些疑问。

即使通过虚指令明白了`c = c++`的与众不同，但是还是不明白为什么会出现这种结果，Java在区分`c = c++`和`d = ++d`不同的时候是根据表达式的优先级来却别的吗？

还有就是相关的官方权威的资料一直没有找到。






