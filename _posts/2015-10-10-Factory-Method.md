---
layout:     post
title:      "设计模式之工厂方法模式"
subtitle:   "Factory Method"
date:       2015-10-10 10:37:33
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 设计模式
---
Author ： [Shinelw](https://github.com/Shinelw)

>我们来假设生活中的一个场景，老师布置下来一项任务，让每一个人做一个简单的手工工艺品。为了完成这样一个即使很简单的工艺品，我们需要使用到纸张、胶水、剪刀等工具和材料。如果这些工具和材料都由我们自己生产出来，那么这个工艺品如果要制作成功，则需要依赖于我们自己制作这些工具和材料的进度，而这些工具和材料制造的复杂程度要远远大于这件工艺品制作的复杂度。其实，我们只是制作这样一个简单的工艺品，完全没有必要因为工具本身的复杂程度而影响工艺品的制作。因此，最好的方法就是这些工具由专门的工厂来制作，我们只需要了解这些工具和材料，直接使用制作工艺品即可。在一个项目中，我们也很经常会遇到像这样的情况，工厂方法模式也就应运而生。


## 一、定义

定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法使一个类的实例化延迟到其子类。所谓的决定并不是模式允许子类本身在运行时做决定，而是指在编写创建者类时，不需知道创建的产品是哪一个，选择了使用哪个子类，就决定了实际创建的产品是什么。

某种程度上，工厂方法模式改变了我们直接用new创建对象的方式，一个很好的开始，意义重大。

适用性：
	
	在以下情况下可以使用工厂方法模式：
	
		·当一个类不知道它做必须创建的对象的类的时候。
		·当一个类希望由它的子类来指定它所创建的对象的时候。
		·当类将创建对象的职责委托给多个帮助子类的某一个，并且希望将哪一个帮助子类是代理者这一信息局部化的时候。

## 二、角色

	·Product：定义工厂方法所创建的对象的接口
	·ConcreteProduct：实现Product接口
	·Creator：抽象工厂
	·ConcreteCreator：重定义工厂方法以返回一个ConcreteProduct实例。
	
## 三、实现

1.定义工厂方法所创建的对象的接口

		public interface Product {
			public void doSomething();
		}
		
2.实现Product的接口
		
		public class ConcreteProduct implements Product {
			public void doSomething() {
				System.out.println("hello world");
			}
		}
		
3.抽象工厂
		
		public interface Creator {
			//定义工厂方法，返回
			public ConcreteProduct factoryMethod();
		}
		
4.具体工厂实现
		
		public class ConcreteCreator implements Creator {
			public ConcreteProduct factoryMethod() {
				return new ConcreteProduct();
			}
		}
		
5.客户端调用
	
		ConcreteCreator factory = new ConcreteCreator();
		ConcreteProudct product = factory.factoryMethod();
		product.doSomething();
		
### 四、Android源码中的运用
	
		//抽象产品
		public interface Runnable {
    		public abstract void run();
		}
 
		//抽象工厂
		public interface ThreadFactory {
   	 		Thread newThread(Runnable r);
		}	
	
下面是具体的实现：

比如AsyncTask类中工厂的具体实现如下

		//工厂实现类
		private static final ThreadFactory sThreadFactory = new ThreadFactory() {
		private final AtomicInteger mCount = new AtomicInteger(1);
		public Thread newThread(Runnable r) {
			return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
			}
		};
		
		//产品类
		public class Thread implements Runnable {
			//具体方法
			//...
		}


>用Factory Method模式创建对象并不一定会让我们的代码更短,实事上往往更长,我们也使用了更多的类,真正的目的在于这样可以灵活的,有弹性的创建不确定的对象.而且,代码的可重用性提高了,客户端的应用简化了,客户程序的代码会大大减少,变的更具可读性。