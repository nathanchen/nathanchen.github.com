---
layout: post
title: "JavaScript垃圾回收"
category: Reading Notes
tags: ["读书", "JavaScript"]
---
{% include JB/setup %}

## JavaScript垃圾回收

### 标记清楚

JavaScript中最常用的垃圾收集方式是标记清楚（mark-and-sweep）。当变量进入环境时，就将这个变量标记为“进入环境”。从逻辑上讲，永远不能释放进入环境的变量所占用的内存，因为只要执行流进入相应的环境，就可能会用到它们。而当变量离开环境时，则将其标记为“离开环境”。

### 引用计数

另一种不太常见的垃圾收集策略叫做引用计数（reference counting）。引用计数的含义是跟踪记录每个值被引用的次数。当声明了一个变量并将一个引用类型值赋给该变量时，则这个值的引用次数就是1.如果同一个值又被赋给另一个变量，则该值的引用次数加1.

有一个严重的问题：循环引用。循环引用指的是对象A中包含一个指向对象B的指针，而对象B中也包含一个指向对象A的引用，例如：

	function problem(){
		var objectA = new Object();
		var objectB = new Object();

		objectA.someOtherObject = objectB;
		objectB.anotherObject = objectA;
	}

在这个例子中，objectA和objectB通过各自的属性相互引用

IE中有一部分并不是原生JavaScript对象。例如，其BOM和DOM中的对象就是使用C++以COM对象的形式实现的，而COM对象的垃圾收集机制采用的就是引用计数策略。

	var element = document.getElementById("some_element");
	var myObject = new Object();
	myObject.element = element;
	element.someObject = myObject;

为了避免类似这种循环引用问题，最好是在不使用它们的时候手工断开

	myObject.element = null;
	element.someObject = null;

### 性能问题

IE的垃圾收集器是根据内存分配量运行的，具体一点就是256个变量，4096个对象（或数组）字面量和数组元素（slot）或者64KB的字符串。达到上述任何一个临界值，垃圾回收器就会运行。

这种实现方式的问题在于，如果一个脚本中包含那么多变量，那么该脚本很可能会在其生命周期中一直保有那么多的变量。而这样一来，垃圾收集器就不得不频繁地运行。结果，由此引发了严重性能问题促使IE7重写了其垃圾收集例程。

### 管理内存

分配给Web浏览器的可用内存数量通常要比分配给桌面程序的少。这样做的目的主要是出于安全方面的考虑，目的是防止运行JavaScript的网页耗尽全部系统内存而导致系统崩溃。





#### Reference

