---
layout: post
author: zhao
title:  Java：Integer的值问题
date:   2015-09-21 10:14:54
categories: Java
---

* content
{:toc}


##前言

>平时忽略的知识点，总是会带来意想不到的惊喜。

##例子

之前学Java的时候经常听说及自动装箱和拆箱的

如下程序，使用自动装箱将基本类型的值赋给包装类型实例时，如果数值的范围在-128~127之间，则输出事true，如果超过这个范围则为false。

~~~
public class IntegerTest {
	public static void main(String[] args){
		
		Integer a = 1;
		Integer b = 1;
		System.out.println(a == b); //true

		Integer c = 128;
		Integer d = 128;
		System.out.println(c == d);	//false
		
		Integer e = new Integer(1);
		Integer f = new Integer(1);
		System.out.println(e == f);	//false
	}
}
~~~

##分析

###自动装箱

####程序

>我承认我又钻牛角尖了，专门通过虚指令看了一下。

一个极其简单的程序，就是为了证明Java在自动装箱时的确调用了valueOf()方法。

~~~
public class IntegerTest {
    public static void main(String[] args){
		Integer a = 1;
	}
}
~~~

####虚指令

~~~
 public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=1, locals=2, args_size=1
         0: iconst_1      
         1: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
         4: astore_1      
         5: return        
      LineNumberTable:
        line 5: 0
        line 6: 5
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0       6     0  args   [Ljava/lang/String;
               5       1     1     a   Ljava/lang/Integer;
~~~

从上面可以看出，我在程序中并没有显示地调用valueOf，但是Java虚拟机在执行的时候却调用了这个方法。具体是在哪里控制这个流程，我也很好奇，只是一直没有找到。


###源码

java的包装类的源码在java.lang.Integer中。JDK1.5开始，Integer对象的赋值会自动调用Integer类的valueOf(int i)方法。

网上关于这两个方法的讲解看的我有点迷惑，他们的方法内容明显都比我这个简单，我在jdk1.6和1.7源码中看到的大概都是如下代码。

在valueOf(int i)可以看到，如果i值在某个范围之内的时候，直接返回IntegerCache.cache[]数组中的某个值，如果超过某个范围则new一个新的Integer对象。

~~~
	public static Integer valueOf(int i) {
        assert IntegerCache.high >= 127;
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
~~~

在IntegerCache中，可以看出low的值的-128，high在初始化的时候是先赋值127，不太清楚`sun.misc.VM.getSavedProperty`的作用，暂时没有细看。

总得来说，在cache数组中存放了-128~127这些整数。

~~~
	private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
        }

        private IntegerCache() {}
    }
~~~

##总结

通过上面的分析可以抓住Java在自动装箱时的一个大概流程（以Integer为例）：

1. 先调用Integer i = Integer.valueOf(n)

2. 判断n的范围

3. 如果n在-128~127之间，则将i指向JVM缓存的常量

4. 如果n超出这个范围，就new一个新的Integer对象（这就造成了地址的不同）



