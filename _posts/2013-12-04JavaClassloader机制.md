---
layout: post
title: "Java Classloader机制"
category: Reading Notes
tags: ["读文章", “Java”]
---
{% include JB/setup %}

## Java Classloader机制

![](/img/java_classloader1.png)

Java中`ClassLoader`的加载采用了双亲委托机制，采用双亲委托机制加载类的时候采用如下的几个步骤：

- 当前`ClassLoader`首先从自己已经加载的类中查询是否此类已经加载，如果已经加载则直接返回原来已经加载的类

	每个类加载器都有自己的加载缓存，当一个类被加载了以后就会放入缓存，等下次加载的时候就可以直接返回了。

- 当前`ClassLoader`的缓存中没有找到被加载的类的时候，委托父类加载器去加载，父类加载器采用同样的策略，首先查看自己的缓存，然后委托父类的父类去加载，一直到`bootstrap ClassLoader`

- 当所有的父类加载器都没有加载的时候，再由当前的类加载器加载，并将其放入它自己的缓存中，以便下次有加载请求的时候直接返回。

##### 命名空间

要确定一个类，需要类的全限定名以及加载此类的`ClassLoader`来共同确定。也就是说即使两个类的全限定名是相同的，但是因为不同的`ClassLoader`加载了此类，那么在JVM中它是不同的类。

采用了委托模型以后加大了不同的`ClassLoader`的交互能力，比如`hashmap`，`linkedlist`等，这些类由`bootstrap`类加载器加载了以后，无论你程序中有多少个类加载器，这些类都是可以共享的，这样就避免了不同的类加载器加载了同样名字的不同类以后造成的混乱。

#### 自定义ClassLoader

##### loadClass方法

	public Class<?> loadClass(String name) throws ClassNotFoundException {
	 	 return loadClass(name, false);
	}

	protected synchronized Class<?> loadClass(String name, boolean resolve)
	  throws ClassNotFoundException
	{
	  	// First, check if the class has already been loaded
	  	Class c = findLoadedClass(name);//检查class是否已经被加载过了
	  	if (c == null) {
		      try {
		      if (parent != null) {
		          c = parent.loadClass(name, false); //如果没有被加载，且指定了父类加载器，则委托父加载器加载。
		      } else {
		          c = findBootstrapClass0(name);//如果没有父类加载器，则委托bootstrap加载器加载
		      }
		      } catch (ClassNotFoundException e) {
		          // If still not found, then invoke findClass in order
		          // to find the class.
		          c = findClass(name);//如果父类加载没有加载到，则通过自己的findClass来加载。
		      }
		  }
		  if (resolve) {
		      resolveClass(c);
		  }
		  return c;
	}

`public Class<?> loadClass(String name) throws ClassNotFoundException`没有被标记为`final`，也就意味着我们是可以override这个方法的，也就是说双亲委托机制是可以打破的

##### findClass

`java.lang.ClassLoader`的源代码，我们发现`findClass`的实现如下：

	protected Class<?> findClass(String name) throws ClassNotFoundException {
	  throw new ClassNotFoundException(name);
	}

此方法默认的实现是直接抛出异常，我们可以override这个方法。

**我们在写自己的ClassLoader的时候，如果想遵循双亲委托机制，则只需要override findClass**

##### defineClass

`defineClass`的源代码：

	protected final Class<?> defineClass(String name, byte[] b, int off, int len)
	  throws ClassFormatError{
	      return defineClass(name, b, off, len, null);
	}


从上面的代码我们看出此方法被定义为了`final`，这也就意味着此方法不能被Override，其实这也是jvm留给我们的唯一的入口，通过这个唯一的入口，j**vm保证了类文件必须符合Java虚拟机规范规定的类的定义。此方法最后会调用native的方法来实现真正的类的加载工作。**

你完全可以自己写一个`classLoader`来加载自己写的`java.lang.String`类，但是你会发现也不会加载成功，具体就是因为针对`java.*`开头的类，jvm的实现中已经保证了必须由`bootstrp`来加载。
























#### Reference：
http://imtiger.net/blog/2009/11/09/java-classloader/