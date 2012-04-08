---
layout: post
title: "Design Patterns"
category: 
tags: []
---
{% include JB/setup %}

## Adapter
保持系统的多态性。让第三方接口符合系统已定义的的接口。
你把数据给我，我来带你转给第三方办，你不要关心我是怎么办到的，你不关心我找谁办的，反正肯定给你办成。


我的系统里已经实现了点、线、正方形，现在客户要求我们实现一个圆形
我们会建立一个Circle类来继承Shape，然后需要去实现display、fill、undisplay方法
但是这时我发现同事已经实现了一个画圆的类，但是他的方法名为displayIt、fillIt、undisplayIt
我们不能直接使用这个类，因为那样无法保持多态性，

	class Circle extends Shape
	{
	    private XXCircle pxc; 
	    
	    public Circle()
	    { 
	        pxc = new XXCircle(); 
	    } 

	    public void display()
	    { 
	        pxc.displayIt(); 
	    } 
	} 

## Abstract Factory

	public interface Creator
　　	{
	　　public ProductA factoryA();
	　　public ProductB factoryB();
　	}

　　具体工厂类1：
<<<<<<< HEAD

=======
　　
>>>>>>> 39642d3f0d090f11deccca7907cd39b2ed8d6431
　　public class ConcreteCreator1 implements Creator
　　{
	　　public ProductA factoryA()
	　　{
			return new ProductA1();
	　　}

	　　public ProductB factoryB()
	　　{
			return new ProductB1();
	　　}
　　}

　　具体工厂类2：
<<<<<<< HEAD

=======
　　
>>>>>>> 39642d3f0d090f11deccca7907cd39b2ed8d6431
　　public class ConcreteCreator2 implements Creator
　　{
	　　public ProductA factoryA()
	　　{
			return new ProductA2();
	　　}

	　　public ProductB factoryB()
	   {
		　　return new ProductB2();
	　　}
　　}

<<<<<<< HEAD
Button--->UnixButton/WinButton
Text----->UnixText/WinText
Unix产品族和Windows产品族，不会同时使用。
Factory--->UnixFactory/WinFactory
=======
    Button--->UnixButton/WinButton
　　Text----->UnixText/WinText
　　Unix产品族和Windows产品族，不会同时使用。
　　Factory--->UnixFactory/WinFactory
>>>>>>> 39642d3f0d090f11deccca7907cd39b2ed8d6431

	//两种抽象产品：水果、蔬菜
　　public interface Fruit
　　{

　　}
　　public interface Veggie
　　{

　　}
　　//四种具体产品：北方水果，热带水果，北方蔬菜，热带蔬菜
　　//Northern Fruit
　　public class NorthernFruit implements Fruit
　　{
	　　private String name;
	　　public NorthernFruit(String name)
	　　{

	　　}
	　　public String getName()
	　　{
			return name;
	　　}
	　　public void setName(String name)
	　　{
			this.name = name;
	　　}
　　}
　　//TropicalFruit
　　public class TropicalFruit implements Fruit
　　{
	　　private String name;
	　　public TropicalFruit(String name)
	　　{

	　　}
	　　public String getName()
	　　{
		　　return name;
	　　}
	　　public void setName(String name)
	　　{
		　　this.name = name;
	　　}
　　}
　　//NorthernVeggie
　　public class NorthernVeggie implements Veggie
　　{
	　　private String name;
	　　public NorthernVeggie(String name)
	　　{

	　　}
	　　public String getName()
	　　{
		　　return name;
	　　}
	　　public void setName(String name)
	　　{
		　　this.name = name;
	　　}
　　}
　　//TropicalVeggie
　　public class TropicalVeggie implements Veggie
　　{
	　　private String name;
	　　public TropicalVeggie(String name)
	　　{

	　　}
	　　public String getName()
	　　{
		　　return name;
	　　}
	　　public void setName(String name)
	　　{
		　　this.name = name;
	　　}
　　}
　　//抽象工厂角色
　　public interface Gardener
　　{
	　　public Fruit createFruit(String name);
	　　public Veggie createVeggie(String name);
　　}
　　//具体工厂角色：北方工厂，热带角色
　　public class NorthernGardener implements Gardener
　　{
	　　public Fruit createFruit(String name)
	　　{
		　　return new NorthernFruit(name);
	　　}
	　　public Veggie createVeggie(String name)
	　　{
		　　return new NorthernVeggie(name);
	　　}
　　}
　　public class TropicalGardener implements Gardener
　　{
	　　public Fruit createFruit(String name)
	　　{
		　　return new TropicalFruit(name);
	　　}
	　　public Veggie createVeggie(String name)
	　　{
		　　return new TropicalVeggie(name);
	　　}
　　}


## Facade


## Decorator
要点喝的，有奶茶，咖啡，茶。另外每一种饮品还能加上珍珠，糖，布丁之类的料。

最后我点：奶茶加珍珠加布丁。

	public abstract class Beverage
	{
		String description = "Unknow Beverage";

		public String getDescription()
		{
			return description;
		}
		public abstract double cost();
	}

	public abstract class CondimentDecorator extends Beverage
	{
		public abstract String getDescription();
	}

	public class Espresso extends Beverage
	{
		public Espresso()
		{
			description = "Espresso";
		}
		public double cost()
		{
			return 1.99;
		}
	}

	public class Mocha extends CondimentDecorator
	{
		Beverage beverage;

		public Mocha(Beverage beverage)
		{
			this.beverage = beverage;
		}
		public String getDescription
		{
			return beverage.getDescription + ", Mocha";
		}
		public double cost()
		{
			return .2 + beverage.cost();
		}
	}

	public class Whip extends CondimentDecorator
	{
		Beverage beverage;

		public Mocha(Beverage beverage)
		{
			this.beverage = beverage;
		}
		public String getDescription
		{
			return beverage.getDescription + ", Whip";
		}
		public double cost()
		{
			return .9 + beverage.cost();
		}
	}

	public class StarbuzzCoffee
	{
		public static void main(String args[])
		{
			Beverage beverage = new Espresso();
			beverage = new Mocha(beverage);
			beverage = new Mocha(beverage);
			beverage = new Whip(beverage);

			System.out.println(beverage.getDescription + " $"
				+ beverage.cost());
		}
	}

## Singleton


## Observer

Observer更多的体现的是订阅与发布的思想
举个例子说明，Bea论坛上都会定期发布一些举办活动的通告，而想参加活动的都需要先去注册一下，然后报名
报名后，Bea会给所有的参加者发一封通知信，告诉又有人参加了
当然了，如果你取消注册，那么到时候将收不到通知了。 

	public interface BeaNotice 
	{  
	    public void addObserver(Observer o);  
	    public void removeObserver(Observer o);  
	    public void addUser();  
	}  

	public class BeaNoticeImpl implements BeaNotice
	{  
	    private List observers = new ArrayList();  
	      
	    public void addObserver(Observer o){  
	        observers.add(o);  
	    }  
	      
	    public void removeObserver(Observer o){  
	        observers.remove(o);  
	    }  
	      
	    public void notifyAllObservers(){  
	        for(int i = 0; i < observers.size(); i++){  
	            ((Observer)observers.get(i)).update(this);  
	        }  
	    }  
	      
	    public void addUser(){  
	        //数据库操作，向数据库插入一个用户的信息，然后通知所有注册的用户  
	        notifyAllObservers();  
	    }  
	}  

	public interface Observer 
	{  
	    public void update(BeaNoticeImpl bns);   
	  
	}  

	public class WelcomeLetterObserver implements Observer 
	{  
	    BeaNotice beaNotice = null;  
	      
	    public WelcomeLetterObserver(BeaNotice beaNotice){  
	        this.beaNotice = beaNotice;  
	        beaNotice.addObserver(this);  
	    }  
	      
	    public void update(BeaNoticeImpl bns) {  
	        System.out.println("This is a welocme letter!");  
	    }  
	  
	}  
  
	public class OtherNoticeObserver implements Observer 
	{  
	    BeaNotice beaNotice = null;  
	      
	    public OtherNoticeObserver(BeaNotice beaNotice){  
	        this.beaNotice = beaNotice;  
	        beaNotice.addObserver(this);  
	    }  
	      
	    public void update(BeaNoticeImpl bns) {  
	        System.out.println("there is other Notice!");  
	    }  
	  
	}  

## Strategy


## Proxy
有两个同学B和C，B是男的，C是女的，你晚上想给女同学C打电话咨询点事，
但是你女友不让所以，你只能通过你的男同学B和C通话，
然后把让B把反馈的结果告诉你

	public interface IStudent 
	{  
   		 public String answerSomeQuestion();  
	}  

	public class IGirlImpl implements IStudent 
	{  
	    public String answerSomeQuestion()
	    {  
	        System.out.println("answer some question");  
	        return "this is my answer";  
	    }  
	}  

	public class IBoyProxyImpl implements IStudent 
	{  
	    IGirlImpl girl = null;  
	    public String answerSomeQuestion() 
	    {  
	        System.out.println("do someting before method invoke");  
	        if(girl == null)
	        {  
	            girl = new IGirlImpl();  
	        }  
	        String returnValue = girl.answerSomeQuestion();  
	        System.out.println("do someting after method invoke");  
	        return returnValue;  
	    }  
	}  

## Reference

设计模式之Adapter模式, http://lizwjiang.iteye.com/blog/86391
