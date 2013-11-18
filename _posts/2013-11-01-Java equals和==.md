---
layout: post
title: "Java中的equals和=="
category: Reading Notes
tags: ["读文章", “Java”]
---
{% include JB/setup %}

## equals和==

	StringBuilder s1 = new StringBuilder();
	StringBuilder s2 = new StringBuilder();
	String s = "abc";
	s1.append(s);
	s2.append(s);
	System.out.println("s1.equals(s2):\t\t" + (s1.equals(s2)));
	System.out.println("s1 == s2:\t\t" + (s1 == s2));
	 
	String s3 = s + "1";
	String s4 = s + "1";
	System.out.println("s3 == s4:\t\t" + (s3 == s4));
	System.out.println("s3.equals(s4):\t\t" + (s3.equals(s4)));
	 
	Object obj1 = new Object();
	Object obj2 = new Object();
	System.out.println("obj1.equals(obj2):\t" + (obj1.equals(obj2)));
	System.out.println("obj1 == obj2:\t\t" + (obj1 == obj2));

执行结果如下

	s1.equals(s2):      false
	s1 == s2:       false
	s3 == s4:       false
	s3.equals(s4):      true
	obj1.equals(obj2):  false
	obj1 == obj2:       false

Object类的equals方法的实现：

	public boolean equals(Object obj) {
	    return (this == obj);
	}

Object中的equals就是用==来比较当前对象和传入的参数的。

String的equals实现：

	public boolean equals(Object anObject) {
	    if (this == anObject) {
	        return true;
	    }
	    if (anObject instanceof String) {
	        String anotherString = (String)anObject;
	        int n = count;
	        if (n == anotherString.count) {
	        char v1[] = value;
	        char v2[] = anotherString.value;
	        int i = offset;
	        int j = anotherString.offset;
	        while (n-- != 0) {
	            if (v1[i++] != v2[j++])
	            return false;
	        }
	        return true;
	        }
	    }
	    return false;
	}

它去比较内容了。

**StringBuilder**，在其源码里没有发现equals方法，那么**它就继承了Object的实现，用==来比较传进来的参数。**

### Reference：

http://www.ticmy.com/?p=186