---
layout: post
title: "JavaScript基本包装类型"
category: Reading Notes
tags: ["读书笔记", "JavaScript"]
---
{% include JB/setup %}

	var falseObject = new Boolean(false);
	var result3 = falseObject && true;
	alert(result3); // true
	
	var falseValue = false;
	result4 = falseValue && true;
	alert(result4); // false
	
	alert(typeof falseObject); // object
	alert(typeof falseValue); // boolean
	alert(falseObject instanceof Boolean); // true
	alert(falseValue instanceof Boolean); // false

在这个例子中，我们使用false值创建了一个Boolean对象。然后，将这个对象与基本类型值true构成了逻辑与表达式。在布尔运算中，false && true等于false。可是，**示例中的这行代码是对falseObject而不是对它的值（false）进行求值**。**布尔表达式中的所有对象都会被转换为true**，因此falseObject对象在布尔表达式中代表的是true。

	var num = 10;
	alert(num.toString());
	alert(num.toString(2));
	alert(num.toString(8));
	alert(num.toString(10));
	alert(num.toString(16));

**告诉它返回几进制数值的字符串形式**


#### Reference

