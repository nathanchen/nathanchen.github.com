---
layout: post
title: "Spring协调作用域不同步的bean"
category: Reading Notes
tags: ["读书", "Spring"]
---
{% include JB/setup %}


## 协调作用域不同步的Bean

	<bean id="steelAxe" class="..." scope="prototype"/>

	<bean id="chinese" class="...">
		<lookup-method name="getAxe" bean="steelAxe"/>
	</bean>

chinese是singleton；steelAxe是prototype

程序多次引用chinese的时候，实际引用的是同一个实例

这样会造成steelAxe也每次都是同一个实例，违反了其prototype的scope

#### 解决方法

	public class SteelAxe implements Axe
	{
		public SteelAxe()
		{
			System.out.println("Spring实例化依赖Bean：SteelAxe实例...");
		}
		
		public String chop()
		{
			return "钢斧砍柴！";
		}
	}
	
	public abstract class Chinese implements Person
	{
		public Chinese()
		{
			System.out.println("Spring实例化主调bean：Chinese实例...");
		}
		
		// 交由Spring来实现
		public abstract Axe getAxe();
		
		public void useAxe()
		{
			System.out.println("正在使用 " + getAxe() + "砍柴!");
			System.out.println(getAxe().chop());
		}
	}

实际上，Spring实现getAxe()的逻辑是：

	public Axe getAxe()
	{
		return ctx.getBean("steelAxe");
	}

运行结果：

- Spring实例化主调bean：Chinese实例...
- Spring实例化依赖Bean：SteelAxe实例...
