---
layout: post
title: "Spring Bean的生命周期"
category: Reading Notes
tags: ["读书", "Spring"]
---
{% include JB/setup %}


## Spring Bean生命周期

#### 依赖关系注入之后的行为

	<bean id="chinese" class="..." init-method="init">
		<property name="axe" ref="steelAxe"/>
	</bean>

	public class Chinese implements Person, InitializingBean
	{
		private Axe axe;
		
		public Chinese()
		{
			System.out.println("Spring实例化主调bean：Chinese实例...");
		}
	
		public void setAxe(Axe axe)
		{
			System.out.println("Spring执行依赖关系注入...");
			this.axe = axe;
		}
		
		public void useAxe()
		{
			System.out.println(axe.chop());
		}
		
		public void init()
		{
			System.out.println("正在执行初始化方法 init...");
		}
		
		public void afterPropertiesSet() throws Exception
		{
			System.out.println("正在执行初始化方法 afterPropertiesSet...");
		}
	}

主程序的执行结果如下：

- Spring实例化依赖bean：SteelAxe实例...
- Spring实例化主调bean：Chinese实例...
- Spring执行依赖关系注入...
- 正在执行初始化方法 afterPropertiesSet...
- 正在执行初始化方法 init...
- axe.chop()

#### Bean销毁之前的行为
	
	<bean id="chinese" class="..." close-method="close">
		<property name="axe" ref="steelAxe"/>
	</bean>

	public class Chinese implements Person, DisposableBean
	{
		private Axe axe;
		
		public Chinese()
		{
			System.out.println("Spring实例化主调bean：Chinese实例...");
		}
	
		public void setAxe(Axe axe)
		{
			System.out.println("Spring执行依赖关系注入...");
			this.axe = axe;
		}
		
		public void useAxe()
		{
			System.out.println(axe.chop());
		}
		
		public void close()
		{
			System.out.println("正在执行销毁之前的方法 close...");
		}
		
		public void destroy() throws Exception
		{
			System.out.println("正在执行销毁之前的方法 destroy...");
		}
	}

#### 如果容器中很多Bean都需要指定特定的生命周期行为，则可以利用<bean.../>元素的default-init-method属性和default-destroy-method属性。

![spring中bean生命周期.png](img/spring中bean生命周期.png)