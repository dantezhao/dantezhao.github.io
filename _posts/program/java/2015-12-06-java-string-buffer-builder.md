---
layout: post
title:  "Java：String、StringBuffer和StringBuilder"
categories: 代码之熵
tags: Java
---

* content
{:toc}

## 简介

自己对String的理解总是存在着不同程度的误差，经常处于一知半解的状态，而且对其内部的原理也不是特别清楚，碰巧又和同学聊起这个知识点，秉承爱折腾的原则，在论文答辩之际详细整理一下。




## 说明

最初听说的String、StringBuffer和StringBuilder三者之间的区别主要是下面这个版本（略作总结）：

- String：字符串常量，字符串长度不可变。Java中String是immutable（不可变）的。用于存放字符的数组被声明为final的，因此只能赋值一次，不可再更改。

- StringBuffer：字符串变量（Synchronized，即线程安全）。如果要频繁对字符串内容进行修改，出于效率考虑最好使用StringBuffer，如果想转成String类型，可以调用StringBuffer的toString()方法。

- StringBuilder：字符串变量（非线程安全）。在内部，StringBuilder对象被当作是一个包含字符序列的变长数组。

- 一般情况下,速度从快到慢：StringBuilder > StringBuffer > String。

## 详细分析

下面通过不同的角度来对这三个String相关的类型进行详细的分析和学习，主要通过源码以及反编译的字节码进行学习，另外对于常拿来比较三者之间性能的例子就不再重复了，整理下面内容的主要目标是深入理解这三者的区别。

下面将分别对这三者进行说明，都先从源码中观测一下其创建的过程，以及如何进行添加操作，随后对三个类型做同一种测试，即字符串的拼接，通过虚指令观测其虚拟机层面的执行原理，最后做一个总结。

### String

#### 源码

下面是String在jdk中的源码片段，可以看出，String类中实际存放数据是一个以final 类型的char数组，也就是说该数组是不可变的。

```java

public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
	private final char value[];

	public String() {
		this.value = new char[0];
	}

	public String(char value[]) {
		this.value = Arrays.copyOf(value, value.length);
	}

	public String(String original) {
		this.value = original.value;
		this.hash = original.hash;
	}
}
```

#### 例子

简单代码

```java
public class StringTest {
	public static void main(String[] args) {
		String s = "string ";
		s += "is a good boy!";
		s += "really?";
	}
}
```

从下面的虚指令中可以看出，在往字符串s中添加新内容的时候，其过程在下面代码中注释。

```
[]$ javap -p -v StringTest
	Classfile /home/hadoop/zhdd/StringTest.class
	  Last modified Dec 6, 2015; size 511 bytes
	  MD5 checksum f960070f295b4e22de6bffe8f06634f4
	  Compiled from "StringTest.java"
	public class StringTest
	  SourceFile: "StringTest.java"
	  minor version: 0
	  major version: 51
	  flags: ACC_PUBLIC, ACC_SUPER
	Constant pool:
	   #1 = Methodref          #10.#19        //  java/lang/Object."<init>":()V
	   #2 = String             #20            //  string
	   #3 = Class              #21            //  java/lang/StringBuilder
	   #4 = Methodref          #3.#19         //  java/lang/StringBuilder."<init>":()V
	   #5 = Methodref          #3.#22         //  java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
	   #6 = String             #23            //  is a good boy!
	   #7 = Methodref          #3.#24         //  java/lang/StringBuilder.toString:()Ljava/lang/String;
	   #8 = String             #25            //  really?
	   #9 = Class              #26            //  StringTest
	  #10 = Class              #27            //  java/lang/Object
	  ......
	{
	 ......
	  public static void main(java.lang.String[]);
		flags: ACC_PUBLIC, ACC_STATIC
		Code:
		  stack=2, locals=2, args_size=1
			 0: ldc           #2                  // 将常量池中的“string ”推到栈顶
			 2: astore_1      					  // 再将栈顶的“string ”存入局部变量表
			 3: new           #3                  // new一个StringBuilder
			 6: dup           
			 7: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
			10: aload_1       					  // 将一个局部变量加载到操作栈，即“string”
			11: invokevirtual #5                  // Method java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
			14: ldc           #6                  // String is a good boy!
			16: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
			19: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
			22: astore_1      
			23: new           #3                  // new一个StringBuilder,在这里可以看出来，每次拼接一个数组就要new一个StringBuilder
			26: dup           
			27: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
			30: aload_1       
			31: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
			34: ldc           #8                  // String really?
			36: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
			39: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
			42: astore_1      
			43: return        
		  LineNumberTable:
			line 8: 0
			line 9: 3
			line 10: 23
			line 13: 43
	}
```

### StringBuilder

#### 源码

通过StringBuilder可以看出，StringBuilder继承自AbstractStringBuilder，而且初始化以及appen新的字符串的主要操作都在AbstractStringBuilder中，因此下面主要看一下AbstractStringBuilder的源码。

```java
public final class StringBuilder extends AbstractStringBuilder implements java.io.Serializable, CharSequence {
	public StringBuilder() {
			super(16);
		}
	public StringBuilder(int capacity) {
		super(capacity);
	}
	public StringBuilder append(String str) {
		super.append(str);
		return this;
	}
```

下面是AbstractStringBuilder的部分源码，在源码中可以看出，AbstractStringBuilder在存放数值的也是一个char型的数组，不同的是，没有加final修饰符。

初始化的过程和String类似，在append的时候可以看出，AbstractStringBuilder是类似于扩充数组大小的方式先扩容，再添加进去新的元素。

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
	char[] value;
	int count;
	AbstractStringBuilder(int capacity) {
		value = new char[capacity];
	}
	public AbstractStringBuilder append(String str) {
		if (str == null) str = "null";
		int len = str.length();
		ensureCapacityInternal(count + len);
		str.getChars(0, len, value, count);
		count += len;
		return this;
	}
}
```


#### 例子

下面是StringBuilder拼接字符串的一个简单的例子。

```java
public class StringBuilderTest {
	public static void main(String[] args) {
		StringBuilder sd = new StringBuilder();
		sd.append("Stringbuilder ");
		sd.append("is a good boy!");
    }
}       
```

在虚指令中可以看出，StringBuilder和String不同的是，StringBuilder在append字符串的时候直接拼接就行，不需要每次都new一个新的StringBuilder对象。

```
[hadoop@x129 zhdd]$ javap -p -v StringBuilderTest
......
Constant pool:
   #1 = Methodref          #8.#17         //  java/lang/Object."<init>":()V
   #2 = Class              #18            //  java/lang/StringBuilder
   #3 = Methodref          #2.#17         //  java/lang/StringBuilder."<init>":()V
   #4 = String             #19            //  Stringbuilder
   #5 = Methodref          #2.#20         //  java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #6 = String             #21            //  is a good boy!
   #7 = Class              #22            //  StringBuilderTest
   ......
{
  public StringBuilderTest();
	flags: ACC_PUBLIC
	Code:
	  stack=1, locals=1, args_size=1
		 0: aload_0       
		 1: invokespecial #1                  // Method java/lang/Object."<init>":()V
		 4: return        
	  LineNumberTable:
		line 1: 0
	  public static void main(java.lang.String[]);
	flags: ACC_PUBLIC, ACC_STATIC
	Code:
	  stack=2, locals=2, args_size=1
		 0: new           #2                  // class java/lang/StringBuilder
		 3: dup           
		 4: invokespecial #3                  // Method java/lang/StringBuilder."<init>":()V
		 7: astore_1      
		 8: aload_1       
		 9: ldc           #4                  // String Stringbuilder
		11: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
		14: pop           
		15: aload_1       
		16: ldc           #6                  // String is a good boy!
		18: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
		21: pop           
		22: return        
	 ......
}
```

### StringBuffer

#### 源码

可以看出StringBuffer也是继承自AbstractStringBuilder，而且它的主要操作都是调用super()来操作实现的，唯一不同的是在append等操作的时候添加了synchronized限定，因此是线程安全的。由于StringBuffer和StringBuilder的主要操作都是在父类AbstractStringBuilder中完成的，因此所谓的StringBuilder比StringBuffer的速度快的主要原因应该是synchronized造成的。

```java
public final class StringBuffer extends AbstractStringBuilder implements java.io.Serializable, CharSequence {

    public StringBuffer() {
        super(16);
    }
    public StringBuffer(String str) {
        super(str.length() + 16);
        append(str);
    }

	 public synchronized StringBuffer append(String str) {
        super.append(str);
        return this;
    }
	public synchronized StringBuffer append(StringBuffer sb) {
        super.append(sb);
        return this;
    }
}
```

#### 例子

如下示例基本和StringBuilder的代码相同。

```java
public class StringBufferTest {
	public static void main(String[] args) {
		StringBuffer sb = new StringBuffer();
		sb.append("string buffer:");
		sb.append("is a good boy!");
	}
}
```

在虚指令中看，可以看出，两者的操作基本没有太多区别，不过多解释。

```html

[]$ javap -v -p StringBufferTest
......
Constant pool:
   #1 = Methodref          #8.#17         //  java/lang/Object."<init>":()V
   #2 = Class              #18            //  java/lang/StringBuffer
   #3 = Methodref          #2.#17         //  java/lang/StringBuffer."<init>":()V
   #4 = String             #19            //  string buffer:
   #5 = Methodref          #2.#20         //  java/lang/StringBuffer.append:(Ljava/lang/String;)Ljava/lang/StringBuffer;
   #6 = String             #21            //  is a good boy!
   #7 = Class              #22            //  StringBufferTest
   #8 = Class              #23            //  java/lang/Object
   #9 = Utf8               <init>
 ......
{
  public StringBufferTest();
	flags: ACC_PUBLIC
	Code:
	  stack=1, locals=1, args_size=1
		 0: aload_0       
		 1: invokespecial #1                  // Method java/lang/Object."<init>":()V
		 4: return        
	  LineNumberTable:
		line 1: 0
	  public static void main(java.lang.String[]);
	flags: ACC_PUBLIC, ACC_STATIC
	Code:
	  stack=2, locals=2, args_size=1
		 0: new           #2                  // class java/lang/StringBuffer
		 3: dup           
		 4: invokespecial #3                  // Method java/lang/StringBuffer."<init>":()V
		 7: astore_1      
		 8: aload_1       
		 9: ldc           #4                  // String string buffer:
		11: invokevirtual #5                  // Method java/lang/StringBuffer.append:(Ljava/lang/String;)Ljava/lang/StringBuffer;
		14: pop           
		15: aload_1       
		16: ldc           #6                  // String is a good boy!
		18: invokevirtual #5                  // Method java/lang/StringBuffer.append:(Ljava/lang/String;)Ljava/lang/StringBuffer;
		21: pop           
		22: return        
	  LineNumberTable:
		line 3: 0
		line 4: 8
		line 5: 15
		line 6: 22
}
```


## 总结

通过上面的测试结果可以解释String、StringBuffer和StringBuilder之间的区别。

### 性能问题

速度比较：StringBuilder > StringBuffer > String。

为什么有这样的情况，首先分析StringBuilder > String，这个的主要原因可以在两个例子对比中看出，在String中，每次拼接新的字符串，都会new一个StringBuilder对象，也就是说如果拼接N次，就需要new出来N个StringBuilder对象，这样无疑上速度会慢很多。

再分析StringBuilder > StringBuffer的原因，这个其实已经比较明确，在前文中指出，StringBuffer和StringBuilder的主要不同是StringBuffer加了synchronized修饰，其余的操作都是继承自AbstractStringBuilder父类。

### 线程安全

线程安全就是synchronized的却别，在源码中可以看到。

### 补充

之前理解的时候一直有一个误区，就是在性能的区分上，StringBuilder比String快的原因是StringBuilder没有存放在常量池中而是存放在一些特殊的区域，但是在以上的例子中可以看出，其实在拼接过程中的所有的string都是存放在常量池中的，不同的主要是拼接的方式。


***
2015-12-06 22:14:54
