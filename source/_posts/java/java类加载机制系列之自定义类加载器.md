title: java类加载机制系列之自定义类加载器
date: 2016-05-16 22:08:00
tags:
- java
- 类加载机制

categories: 原创
comments: true
-----


上一篇文章中讲到了Java 类加载器的分类，以及它们各自的作用范围，这篇文章我们来说一下怎么实现自己的类加载器。

<!--more-->
二、自定义类加载器

在进入正文之前，我们先来说一下为什么要实现自己的类加载器，因为对于大部分需求来讲并不需要由自己来实现类加载器，但是对于一些特殊的需求来讲就必须要自己来实现类加载器了，比如当一个类在编译的时候进行了加密操作，在装载到JVM之前，就需要先进行解密操作，然后才能发送。
接下来进入今天的正文，实现自定义的类加载器其实非常简单，只需要继承ClassLoader类，然后重写其中的findClass方法就可以了，下面是我自己写的一个类加载器：

``` java
/**
 * 不同类加载器的加载类型
 * @author peng7
 *
 */
package classLoaderExample;

import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.IOException;

public class MyClassLoader extends ClassLoader {
	private String classByteCodePath;
	public MyClassLoader(String classByteCodePath){
		this.classByteCodePath = classByteCodePath;
	}
	
	@Override
	protected Class<?> findClass(String className)throws ClassNotFoundException{
		byte[] value = null;
		BufferedInputStream in = null;
		try{
			in = new BufferedInputStream(new FileInputStream(classByteCodePath+className+".class"));
			value = new byte[in.available()];
			in.read(value);
		}catch(IOException e){
			e.printStackTrace();
		}finally{
			if(in!=null){
				try {
					in.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
				
		return defineClass(value, 0, value.length);
	}
	public static void main(String args[]){
		MyClassLoader classLoader = new MyClassLoader("E:/java/");
		try {
			System.out.println("当前类的类加载器"
					+classLoader.loadClass("Test").getClassLoader().getClass().getName());
			System.out.println("当前类的父类加载器："+
					classLoader.loadClass("Test").getClassLoader().getParent().getClass().getName());
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}


```
在执行该类之前，先来对其做下分析，从上面的代码中可以看到它有一个构造方法，该构造方法的参数用于指定字节码文件即编译后的类文件的路径，接下来就是findClass方法了，它有一个String类型的参数，该参数用于指定要加载类的全限定名称，接下来就是将指定的字节码文件转换为输入流，并放到一个二进制数组中，如果指定的字节码文件进行了加密操作，在这个过程中还可以做一些解密性的操作，最后调用defineClass就可以获得一个该类的Class实例；如果指定路径不存在，或者指定路径中不存在指定的类，则会抛出ClassNotFoundException。
现在可以运行main方法了，得到如下的输出：

```
当前类的类加载器classLoaderExample.MyClassLoader
当前类的父类加载器：sun.misc.Launcher$AppClassLoader
```
从输出结果中可以看到Test类的类加载器就是我们自定义的MyClassLoader，而MyClassLoader的父类加载器是AppClassLoader。为什么是AppClassLoader而不是ExtClassLoader或者Bootstrap ClassLoader呢，因为ClassLoader位于JAVA_HOME的lib下，上一节我们讲过lib下的类是由AppClassLoader加载的





