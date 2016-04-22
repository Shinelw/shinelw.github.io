---
layout:     post
title:      "设计模式之单例模式"
subtitle:   ""
date:       2015-10-03 15:37:14
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 设计模式
---

Author ： [Shinelw](https://github.com/Shinelw)

>单例其实就是唯一实例的意思，也就是说一个类只有一个唯一的实例。开发人员都知道，在java里，只要new一个类，就会创建这个类的一个实例，如果把这个类new多次，就会创建这个类的多个实例，但是有时候不管new多少次，就只需这个类的一个实例，比如日志记录中的管理类，Android中的InputMethodManager类、Editable类等。


## 一、概念

单例模式确保某个类只有一个实例，而且自行实例化并向整个系统提供这个实例。在计算机系统中，线程池、缓存、日志对象、对话框、打印机、显卡的驱动程序对象常被设计成单例。这些应用都或多或少具有资源管理器的功能。每台计算机可以有若干个打印机，但只能有一个Printer Spooler，以避免两个打印作业同时输出到打印机中。每台计算机可以有若干通信端口，系统应当集中管理这些通信端口，以避免一个通信端口同时被两个请求同时调用。总之，选择单例模式就是为了避免不一致状态。

单例模式特点：
	
	1.单例类只能有一个实例。
	2.单例类必须自己创建自己的唯一实例
	3.单例类必须给所有其他对象提供这一实例
## 二、分类

  *单例模式有很多种写法，*
  
### 1.饿汉式单例模式

```java
		public class Singleton {
			private static Singleton instance = new Singleton();
			private Singleton() {
				//防止通过构造方法获取实例
			}
			public static Singleton getInstance() {
				return instance
			}
		}
```

这里需要注意的是，在单例模式中，需要将构造函数定义为private（私有），以防止通过构造方法获取该类的实例。

### 2.懒汉式单例模式

```java
		public class Singleton {
			private static Single instance;
			private Singleton() {
				//防止通过构造方法获取实例
			}
			public static Singleton getInstance() {
				if(instance == null) {
					instance = new Singleton();
				}
				return instance;
			}
		} 
```

上述的写法为一般的懒汉式，这种写法在单线程中使用是完全没有问题的。但是在多线程中就会出现问题，但多个线程同时都进行***if(instance == null)***判断时，就会产生多个该类的实例，这就违背了单例模式的原则。所以就要将其改成线程安全的懒汉式写法。


```java
		/**懒汉式变种，解决线程安全问题**/ 
		public class Singleton {
			private static Single instance;
			private Singleton() {
				//防止通过构造方法获取实例
			}
			//增加同步机制
			public static synchronized Singleton getInstance() {
				if(instance == null) {
					instance = new Singleton();
				}
				return instance;
			}
		} 
```

但是，将同步机制放在获取实例方法上，会导致程序每获取一次实例，都进入synchronized（同步）机制，如果在程序运行时，需要大量获取该类的实例，这种写法就非常低效。*于是另外又有一种写法，将那个synchronized放在产生实例的代码前，如下：*

```java
		public class Singleton {
			private static Single instance;
			private Singleton() {
				//防止通过构造方法获取实例
			}
			public static Singleton getInstance() {
				//增加同步机制
				if(instance == null) {
					synchronized（Singleton.class）{
					instance = new Singleton();
					}
				}
				return instance;
			}
		} 
```

同时，上述代码虽然避免了每次进入同步机制，但是又存在多线程下返回多个实例的问题。为此产生了另一种写法：双检测锁机制。
### 3.双检测锁机制单例模式
	
```java	
		public class Singleton {
			private static Single instance;
			private Singleton() {
				//防止通过构造方法获取实例
			}
			public static Singleton getInstance() {
				//双检测锁机制
				if(instance == null) {
					synchronized（Singleton.class）{
						if(instance == null) {
							instance = new Singleton();
						}
					}
				}
				return instance;
			}
		} 
```
		
>饿汉式和懒汉式区别:

>饿汉就是类一旦加载，就把单例初始化完成，保证getInstance的时候，单例是已经存在的了，而懒汉比较懒，只有当调用getInstance的时候，才回去初始化这个单例。

>另外从以下两点再区分以下这两种方式：

>1、线程安全：

>饿汉式天生就是线程安全的，可以直接用于多线程而不会出现问题，懒汉式本身是非线程安全的，为了实现线程安全有以上三种写法,这三种实现在资源加载和性能方面有些区别。

>2、资源加载和性能：

>饿汉式在类创建的同时就实例化一个静态对象出来，不管之后会不会使用这个单例，都会占据一定的内存，但是相应的，在第一次调用时速度也会更快，因为其资源已经初始化完成。

>而懒汉式顾名思义，会延迟加载，在第一次使用该单例的时候才会实例化对象出来，第一次调用时要做初始化，如果要做的工作比较多，性能上会有些延迟，之后就和饿汉式一样了。

### 4.内部类的实现

```java
		public class Singleton {
			private Singleton() {
				//防止通过构造方法获取实例
			}
			private static SingletonHolder {
				private static Single instance = new Singleton();
			}
			public static Singleton getInstance() {
				reutrn SingletonHolder.instance;
			}
		} 
```

优点：延迟加载，线程安全（java中class加载时互斥的），也减少了内存消耗

## 三、Android中源码运用

### 1.InputMethodManager类 

```java
		public final class InputMethodManager { 
		static final boolean DEBUG = false; 
		static final String TAG = "InputMethodManager"; 
		static final Object mInstanceSync = new Object(); 
		static InputMethodManager mInstance; 
		final IInputMethodManager mService; 
		final Looper mMainLooper; 
```

创建唯一的实例static InputMethodManager mInstance; 

```java		 
		/** 
		* Retrieve the global InputMethodManager instance, creating it if it 
		* doesn't already exist. 
		* @hide 
		*/ 
		static public InputMethodManager getInstance(Context context) { 
		return getInstance(context.getMainLooper()); 
		} 
		/** 
		* Internally, the input method manager can't be context-dependent, so 
		* we have this here for the places that need it. 
		* @hide 
		*/ 
		static public InputMethodManager getInstance(Looper mainLooper) { 
		synchronized (mInstanceSync) { 
		if (mInstance != null) { 
		return mInstance; 
		} 
		IBinder b = ServiceManager.getService(Context.INPUT_METHOD_SERVICE); 
		IInputMethodManager service = IInputMethodManager.Stub.asInterface(b); 
		mInstance = new InputMethodManager(service, mainLooper); 
		} 
		return mInstance; 
		} 
```

防止多线程同时创建实例： 

```java		
		synchronized (mInstanceSync) { 
		if (mInstance != null) { 
			return mInstance; 
			}
		} 
```

当没有创建实例对象时，调用*mInstance = new InputMethodManager(service, mainLooper)*; 
其中类构造函数如下所示

```java
		InputMethodManager(IInputMethodManager service, Looper looper) { 
		mService = service; 
		mMainLooper = looper; 
		mH = new H(looper); 
		mIInputContext = new ControlledInputConnectionWrapper(looper, 
		mDummyInputConnection); 
		if (mInstance == null) { 
		mInstance = this; 
		} 
		}
```
		
### 2.AccessibilityManager类

```java
		public static AccessibilityManager getInstance(Context context) {  
    		synchronized (sInstanceSync) {  
        		if (sInstance == null) {  
            		sInstance = new AccessibilityManager(context);  
       				 }  
    		}  
    			return sInstance;  
		}
```

### 3.Editable类

```java
		private static Editable.Factory sInstance = new Editable.Factory();  
		/** 
		 * Returns the standard Editable Factory. 
 		*/  
		public static Editable.Factory getInstance() {  
   		 return sInstance;  
		}  
```		
		
