---
layout: post
author: zhao
title: Java：详解传值和传引用
modified: 2015-08-17
tags: [Java]
image:
  feature: pic-8.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

##传值和传引用

> When you're passing primitives into a method ,you get a distinct copy of the primitive. When you're passing a reference into a method ,  you get a copy of the reference.

以上引自《Thinging in Java》，总结一下就是不管Java参数的类型是什么，一律传递参数的副本。

在Java中，变量分为以下两类：

1. 对于基本类型变量（int、long、double、float、byte、boolean、char），Java是传值的副本。

2. 对于一切对象型变量，Java都是传引用的副本，其实穿引用副本的实质就是复制指向地址的指针。

##一、传值

###例1：传Int类型的值

####程序
{% highlight java linenos %}
	@Test
	public void passInt(){		
		int testInt = 0;
		System.out.println("Before operation, testInt is : " + testInt);
		intOperation(testInt);
		System.out.println("After operation, testInt is : " + testInt);
	}

	public void intOperation(int i){		
		i = 5;
		System.out.println("In operation, testInt is : " + i);
	}
{% endhighlight %}

####结果

{% highlight python %}
Before operation, testInt is : 0

In operation, testInt is : 5

After operation, testInt is : 0
{% endhighlight %}

####总结

结果不难看出来，虽说intOperation()方法改变了传进来的参数值，但对这个参数源本身并没有影响，参数类型是简单类型的时候，是按值传递的。以参数类型传递简单类型的变量时，实际上是将参数的值作为一个副本传进方法函数的，那么在方法函数中不管怎么改变值，其**结果都是只改变了副本的值，而不是源值**。

##二、传引用

###例2：传对象型变量

####代码
{% highlight java linenos %}
	@Test
	public void passClass(){
		Person person = new Person(175,140);
		
		System.out.println("Before classOperation, the person height is " + person.height +
				" and the weight is " + person.weight);
		
		classOperation(person);
		
		System.out.println("After classOperation, the person height is " + person.height +
				" and the weight is " + person.weight);
	}
	
	public void classOperation(Person person){
		person.height = 190;
		person.weight = 160;
		
		System.out.println("In classOperation, the person height is " + person.height +
				" and the weight is " + person.weight);
		
	}
	

	public class Person{
		
		int height;
		int weight;
	
		public Person(int hei, int wei){
			this.height = hei;
			this.weight = wei;
		}
	}
{% endhighlight %}

####结果

{% highlight python %}
Before classOperation, the person height is 175 and the weight is 140

In classOperation, the person height is 190 and the weight is 160

After classOperation, the person height is 190 and the weight is 160
{% endhighlight %}

####总结

从结果不难看出，在经过classOperation()处理之后，person的值发生了变化。

传引用和之前的传值是有一些区别的，可以这样理解：

- 传值：当前的值好比是一个苹果A，在Java中的传值就相当于是把这个苹果复制了一个苹果B，实际上传递的其实是苹果B，也就是说不管对苹果B做出怎样的修改，苹果A是不变的。

- 传引用：可以这样理解，当前的person是一个仓库，因为这个仓库（对象型变量）不像基本类型变量那么小，就好比一个仓库比一个苹果大的多的多一样。聪明的Java不会每次传递这个仓库的时候就重新复制一份仓库传过去，它的做法是把钥匙A复制一把钥匙B（钥匙就相当于引用，引用指向仓库），传递的时候把钥匙B传递过去，和传值类似的地方在于钥匙B怎么改变都不影响钥匙A这个引用的值，但是钥匙A和钥匙B都指向同一个仓库，也就是说通过钥匙B来改变了这个仓库的值（比如偷取走一万吨粮食），那么这个仓库的值就确确实实改变了，如果再通过钥匙A访问这个仓库，得到的结果和B访问时一样的（少了一万吨粮食）。


###例3：传String类型

####代码

{% highlight java linenos %}
	@Test
	public void passString(){
		String testString = "Hello";
		System.out.println("Before operation, testString is : " + testString);
		stringOperation(testString);
		System.out.println("After operation, testString is : " + testString);
		
	}
	
	public void stringOperation(String s){
		s = "World";
		System.out.println("In operation, testString is : " + s);
	}
{% endhighlight %}

####结果

{% highlight python %}
Before operation, testString is : Hello

In operation, testString is : World

After operation, testString is : Hello
{% endhighlight %}

####总结

String类型也是对象型类型，所以它是传引用副本。

**但是问题来了，String类型也是对象型类型，那么为什么String类型的对象，经过stringOperation()后，值没有发生变化？**

首先String类型是对象型变量，所以它是传引用的副本。不要因为String在Java里面非常易于使用而且不需要new，就认为String是基本变量类型。只不过String是一个非可变类，使得其传值还是穿引用显得没什么区别。

然后来解决上面说的问题，其实问题是出在String的创建上面了，在stringOperation()中，`s = "World";`这条语句相当于执行了`String s = new String("World");`，对，也就是说，在这个方法中的`s`相当于是一个新的String对象，当这个函数结束时，`s`作用消失，原来内存地址的值没有变化。

下面来具两个例子来说明这个问题：

###例4：传String类型——使用new来创建新对象

####代码
{% highlight java linenos %}
	@Test
	public void passString(){
		String testString = new String("Hello");
		System.out.println("Before operation, testString is : " + testString);
		stringOperation(testString);
		System.out.println("After operation, testString is : " + testString);
		
	}
	
	public void stringOperation(String s){
		s = new String("World");
		System.out.println("In operation, testString is : " + s);
	}
{% endhighlight %}

####输出

{% highlight python %}
Before operation, testString is : Hello

In operation, testString is : World

After operation, testString is : Hello

{% endhighlight %}

####总结

例4和例3的区别就在于，创建String都使用了new，最后的输出结果是不一样的。

###例5：传对象型变量——在classOperation()使用new创建

####代码
{% highlight java linenos %}
	public class Person{
		
		int height;
		int weight;
	
		public Person(int hei, int wei){
			this.height = hei;
			this.weight = wei;
		}
	}
	
	@Test
	public void passClass2(){
		
		Person person = new Person(175,140);
		
		System.out.println("Before classOperation, the person height is " + person.height +
				" and the weight is " + person.weight);
		
		classOperation2(person);
		
		System.out.println("After classOperation, the person height is " + person.height +
				" and the weight is " + person.weight);
	}
	
	public void classOperation2(Person person){
		
		person = new Person(190,160);
		
		System.out.println("In classOperation, the person height is " + person.height +
				" and the weight is " + person.weight);
		
	}
{% endhighlight %}

####结果

{% highlight python %}
Before classOperation, the person height is 175 and the weight is 140

In classOperation, the person height is 190 and the weight is 160

After classOperation, the person height is 175 and the weight is 140
{% endhighlight %}

####总结

最后一个例子其实就是为了说明一个情况，例3的stringOperation()中看到了字符串的赋值操作其实就相当于例5中classOperation2()里面new出来的新对象。


##三、总结

上面的分析主要是基于结果来判断的，可能不是特别准确，望批评指正。