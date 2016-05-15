title: java类加载机制系列之类加载器
date: 2016-05-15 21:08:00
tags:
- 正则表达式

categories: 原创
comments: true
-----


前几天从同事（[Hassan](http://blog.fenxiangz.com/)）那里看到了一个Java类加载机制的问题，一看之下发现自己对Java类的加载过程还是模棱两可，不胜惶恐，今天花了三个多小时的时间将之前入手的一本Java虚拟机精讲中的类加载机制详细看了一遍，所以从今天起我将每天写一篇关于类加载机制方面的学习成果。

<!--more-->
一、类加载器
在讲述类加载机制之前，很有必要先说一下什么是类加载器，因为整个类加载机制就是由类加载开始的。类加载器的主要任务就是根据类的全路径名读取该类的二进制字节流，然后输入到Java虚拟机内部，其主要分为以下三种：
1、Bootstrap ClassLoader
2、ExtClassLoader;
3、AppClassLoader;

其中Bootstrap ClassLoader负责加载 JAVA_HOME/lib下的类，它是由C++编写而成嵌套在JVM内部。而ExtClassLoader和AppClassLoader都是使用Java本身的语言实现的，ExtClassLoader负责加载JAVA_HOME/lib/ext下的类，而AppClassLoader主要负责加载用户classPath下的类。下面这个例子很好的展示了这三者的作用类型：

``` java
/**
 * 不同类加载器的加载类型
 * @author peng7
 *
 */
public class ClassLoaderTest {

	public static void main(String[] args) {
		ClassLoader loader = System.class.getClassLoader();
		System.out.println(loader!=null?loader.getClass().getName():null);
		
		System.out.println(CollationData_ar.class.getClassLoader().getClass().getName());
		
		System.out.println(ClassLoaderTest.class.getClassLoader().getClass().getName());
	}

}
```
执行结果如下：

```
null
sun.misc.Launcher$ExtClassLoader
sun.misc.Launcher$AppClassLoader

```
可以看到上面输出结果的第一行为null，java.lang.System类在JAVA_HOME/lib下，由Bootstrap ClassLoader加载，而Bootstrap ClassLoader 由C++实现，所以输出为null；而 sun.text.resources.CollationData_ar位于JAVA_HOME/lib/ext下，所以由ExtClassLoader类加载；ClassLoaderTest是我自己定义的一个类，在classPath下，由AppClassLoader负责加载。当然，由于一些特殊需求，我们也可以定义自己的ClassLoader，在下一篇中我们将详细介绍如何自己定义ClassLoader。



