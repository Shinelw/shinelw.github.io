---
layout:     post
title:      "谈谈Kotlin特性在开发中的应用"
subtitle:   "Kotlin - Apply In Coding"
date:       2017-03-17 17:51:57
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Kotlin
---


## 概述
Kotlin是JetBrains团队开发的一种静态类型的函数式编程语言，它基于Java虚拟机（JVM），通过Kotlin的编译器生成的JVM字节码与Java编译的字节码非常相似。事实也的确如此，虽然在语法方面与Java差异很大，但是Kotlin和Java在开发过程中是完全兼容的。所以说，Kotlin很值得一试。本文将重点谈谈Kotlin新特性在实际项目开发中的感受。

关于语法使用及入门可以参见[官方文档](https://kotlinlang.org/docs/reference/)或者[中文文档](https://kotlin-zhcn.github.io/docs/reference/)。

## 简洁，使用更少的代码做更多的事
在我看来，Kotlin很关键的一个优点就是简洁。相对于Java，使用Kotlin往往能够用更少的代码获得更多的功能。这有什么好处呢？很简单，写的代码越少，逻辑越清晰，所犯的错误就会越少。
### 1. 数据类
数据类大量重复的getter和setter相信会是很多人在开发过程中吐槽的一个点。举一个很经典的例子，我们需要一个Person的数据类。

在Java中，需要这么写：

```java
public class Person {
    private String name;
    private int age;
    private int sex;
    private float height;

    public Person(String name, int age, int sex, float height) {
        this.name = name;
        this.sex = sex;
        this.age = age;
        this.height = height;
    }
    
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
    
    public int getSex() {
    	 return sex;
    }
    
    public void setSex(int sex) {
    	this.sex = sex;
    }

    public float getHeight() {
        return height;
    }

    public void setHeight(float height) {
        this.height = height;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", sex=" + (sex == 0 ? "男" : "女") + 
                ", height=" + height +
                '}';
 }
```

在Kotlin里，我们只需要一行代码就能完成以上的功能：

```kotlin
data class Person(var name: String,
					 var age: Int,
					 var sex: Int,
					 var height: Float)
```
Kotlin提供的数据类会让你自动获得所需的getter、setters、toString()，这很大程度上减少了大量重复的工作。当然，我们也可以很轻松的去覆盖这些函数，做自定义的事情，但是在大多数情况下，只需声明类和属性就已经足够了。

### 2. 区间表达式
在Java中我们经常要写这样的代码，
```java
for(int i = 0; i <= 10; i++){
	System.out.println(i)
}
```
但是在Kotlin中，支持**..**操作符形式的区间表达式，我们转换成Kotlin就变成了这样：
```kotlin
for(i in 0..10){
	println(i)
}
```
是不是简洁优雅很多，不仅如此，还有更多相关的功能。

```kotlin
//倒序迭代
for(i in 10 downTo 0){
	...
}

//步长为2的迭代
for(i in 0..10 setp 2){
	...
}

//i在[0,10)区间，排除了10
for(i in 0 until 10){
	...
}
```

### 3. Lamda表达式
Java在Java8才支持Lamda语法，在Kotlin里完全支持。有对比才有差距，看例子：

**Java中：**

```java
btn.setOnClickListener(new View.OnClickListener() {
     @Override
     public void onClick(View v) {
         //dosomething
}});
```

**Kotlin中：**

```kotlin
btn.setOnClickListener { //dosomething }
```

### 4. 类扩展
类扩展是一个超级强大的功能，我终于可以摆脱大量的Util工具类了。：）

看一个例子：

```kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) { 
	val tmp = this[index1] // 'this' 对应该列表 
	this[index1] = this[index2]	this[index2] = tmp	}
}

//使用
val l = mutableListOf(1, 2, 3)l.swap(0, 2) // 'swap()' 内部的 'this' 得到 'l' 的值
```
是不是功能强大且代码简洁优雅？当然，扩展并不能真正的修改它所扩展的类。通过定义一个扩展，我们并没有在一个类中插入新的方法，仅仅是可以通过该类型的变量用点表达式来调用这个新函数。
口说无凭，我们来看看Kotlin编译后的字节码：

定义：

```
 public final class ExtensionKt {
  // access flags 0x19
  // signature (Ljava/util/List<Ljava/lang/Integer;>;II)V
  // declaration: void swap(java.util.List<java.lang.Integer>, int, int)
  public final static swap(Ljava/util/List;II)V
    @Lorg/jetbrains/annotations/NotNull;() // invisible, parameter 0
   L0
    ALOAD 0
    LDC "$receiver"
    INVOKESTATIC kotlin/jvm/internal/Intrinsics.checkParameterIsNotNull (Ljava/lang/Object;Ljava/lang/String;)V
   L1
    LINENUMBER 6 L1
    ALOAD 0
    ILOAD 1
    INVOKEINTERFACE java/util/List.get (I)Ljava/lang/Object;
    CHECKCAST java/lang/Number
    INVOKEVIRTUAL java/lang/Number.intValue ()I
    ISTORE 3
   L2
    LINENUMBER 7 L2
    ALOAD 0
    ILOAD 1
    ALOAD 0
    ILOAD 2
    INVOKEINTERFACE java/util/List.get (I)Ljava/lang/Object;
    INVOKEINTERFACE java/util/List.set (ILjava/lang/Object;)Ljava/lang/Object;
    POP
   L3
    LINENUMBER 8 L3
    ALOAD 0
    ILOAD 2
    ILOAD 3
    INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer;
    INVOKEINTERFACE java/util/List.set (ILjava/lang/Object;)Ljava/lang/Object;
    POP
   L4
    LINENUMBER 9 L4
    RETURN
   L5
    LOCALVARIABLE tmp I L2 L5 3
    LOCALVARIABLE $receiver Ljava/util/List; L0 L5 0
    LOCALVARIABLE index1 I L0 L5 1
    LOCALVARIABLE index2 I L0 L5 2
    MAXSTACK = 4
    MAXLOCALS = 4
    
  // compiled from: extension.kt
}

```

使用：

```
   L1
    LINENUMBER 11 L1
    ALOAD 1
    ICONST_0
    ICONST_2
    INVOKESTATIC ExtensionKt.swap (Ljava/util/List;II)V
   L2
```

可以看出，Kotlin在编译时会以所在文件名extension创建静态类ExtensionKt，并将定义的扩展方法swap编译成静态方法供外部调用。

在Java中形式大致是这样：

```java
public final class ExtensionKt {
   public static final void swap(@NotNull List list, int index1, int index2) {
      int tmp = ((Number)list.get(index1)).intValue();
      list.set(index1, receiver.get(index2));
      list.set(index2, Integer.valueOf(tmp));
   }
}
```

所以在底层，其实就是我们平时所写的Util工具类，但是Kotlin默认帮我们实现了，我们只需更简洁的编写调用就好了。

## 安全，避免NPE空指针异常

### 1. 空安全
这是一个让人又爱又恨的特性。一直以来，NullPointException空指针异常在开发中是最低级也最致命的问题。我们往往需要进行各种null的判断以试图去避免NPE的发生。Kotlin基于这个问题，提出了一个空安全的概念，即每个属性默认不可为null。

举个例子。

```kotlin
var a: String = "abcd"
a = null //编译错误
```
如果要允许为空，我们需要手动声明一个变量为可空字符串类型，写为**String?**

```kotlin
var a: String? = "abcd"
a = null //编译成功
```
那怎么实现默认不可为Null呢？
我们来看一下字节码:

```
  @Lorg/jetbrains/annotations/NotNull;() // invisible
   L0
    LINENUMBER 10 L0
    GETSTATIC PersonKt.a : Ljava/lang/String;
    ARETURN
   L1
   
   //init
   static <clinit>()V
   L0
    LINENUMBER 10 L0
    LDC "abcd"
    PUTSTATIC PersonKt.a : Ljava/lang/String;
    RETURN
    MAXSTACK = 1
    MAXLOCALS = 0
```

可以看到Kotlin对变量进行了NotNull注解。翻译成Java代码：

```
@NotNull
String a = "abcd"
```

不仅如此，为了避免NPE异常，Kotlin做了一件很有趣的事：当你允许属性可空时，Kotlin编译器将不允许你在未经检查的情况下引用它。

```kotlin
var person: Person? = null
person.name = "shinelw"  //编译失败
person?.name = "shinelw" //编译成功
```

如上面的代码所示，当person对象可为null时，必须强制使用**?.**来进行null检查。看看**?.**在字节码里的样子，

```
    LINENUMBER 13 L1
    ALOAD 0
    DUP
    IFNULL L2
    LDC "shinelw"
    INVOKEVIRTUAL Person.setName (Ljava/lang/String;)V
    GOTO L3
   L2
    POP
   L3
```

可见，在person获取name属性的时候进行了null的判断，翻译成java代码：

```java
Person person = (Person)null;
if(person != null) {
   person.setName("shinelw");
}
```

这么看来，空安全特性的确带来了巨大的好处，极大程度上避免了空指针异常的出现。*然而*，世上没有十全十美的东西。空安全在开发过程中给我带来了很多幸福的烦恼。举个例子，以前用Java是这样的：

```java
public class A {
	String a;
	String b;
	String c;
}
```
现在呢，Kotlin中是这样的：

```kotlin
class A {
	var a: String? = null
	var b: String? = null
	var c: String? = null
}
```
看出区别了吗？

在Kotlin中我们需要在定义变量是就必须给出初始值。开发过程中，很多情况下变量在定义时尚不知道要赋何值的，Kotlin强制初始化赋值让整个代码看起来很“怪异”。对我来说，如果一个变量可为null时，它应该是隐含地就默认给予了null值。

我希望应该是这样的，

```
class A {
	var a: String?  //默认值为null
	var b: String?  //默认值为null
	var c: String?  //默认值为null
}
```

虽然说Kotlin提供了**lateinit**类型懒加载的方式进行初始化，但是也并不能很好的支持全部情况，它只能用于**var**的属性，并且只能在属性没有自定义getter或者setter时候使用。属性的类型必须是非空值，并且它不能使原始类型。

当然，我们换个角度，从语言设计的角度来说，Kotlin这么设计又是很合理的。所有属性要求强制显式的初始化能够更容易的推理代码，明确每个属性在何时何地初始化。

总的来说，空安全机制所做的事情就是，让我们原本在逻辑代码中进行大量判空的工作转移到了初始化上，并很大程度地减轻了工作量。

## 更优雅，遵循Effective Java设计
### 1. 类默认不可继承
《Effective Java》提出一种观点：组合优于继承，避免滥用继承。要么为继承而设计，并提供文档说明，要么就禁止继承。至于为什么这么说呢，我见过一句话很形象：摊开来的代码，比叠起来的代码，更加一目了然。详细可以自行阅读《Effective Java》。

Kotlin默认类是final类型的，即每个类默认不可继承。只有你真的需要继承的时候，再通过**open**声明使用，声明方式如下：

```kotlin
open class A
```

### 2. 更有效地使用构建器模式
我们建议使用构建器模式，当Java的构造器存在多个可选的参数时，情况就会变得很复杂，代码冗长，也更容易出错。Kotlin提供了一种更有效的构造器方式，通过默认参数的功能实现。

```kotlin
data class Person(var name: String, 
					 var age: Int = 18,
					 var sex: Int = 0,
					 var height: Float = 1.8f
					 var weight: Float = 60f)

//创建对象
val person: Person = Person(name="shinelw",
								 age = 10,
								 height = 1.7f)
```


### 3. 单例模式
Kotlin默认提供了单例模式的模板，通过**object**关键字即可实现。

```kotlin
object Singleton {
	//各种函数
	fun a(){...}
	...
}

//使用时
Singleton.a()
```
完全不需要手动构建，看上去很好。但是我有一点质疑，它是不是线程安全的，是不是懒加载的。随后通过Kotlin编译器得到字节码，然后再反编译回Java代码。是长这样子的：

```java
public class Singleton {
   public static final Singleton INSTANCE;

   public final void a() {
   }

   private Singleton() {
      INSTANCE = (Singleton)this;
   }

   static {
      new Singleton();
   }
}
```
糟糕的是，从上面代码可以看出，Kotlin的**object**只是一个最简单的饿汉式的单例模式。在第一次加载类到内存的时候就会初始化，虽然它是线程安全的，但是不完美，对吗？

如果你是一个追求完美的人，下面是类似于静态内部类方式实现的单例模式，懒加载且线程安全。缺点是跟Java一样，需要手动构建。：)

```kotlin
class Singleton private constructor(){
    companion object {
        fun getInstance(): Singleton {
            return SingletonHolder.instance
        }
    }
    private object SingletonHolder {
        val instance: Singleton = Singleton()
    }
    
    //各种函数..
    fun a(){}
}
```

### 4. 重载必须使用override

Java中对于重载的注解@Override不是强制的，一旦项目代码很复杂，这将是一场灾难。当你分不清哪些是重载方法时，对方法进行参数修改是灾难性的。Kotlin基于这点，要求重载方法时必须加上**override**关键字。如果没写，编译器将会报错，强制你加上。

```kotlin
override fun a(){...}
```

## 完全兼容，与Java互操作
这是Kotlin与Scala相比，优势突出的一点。我们可以在Kotlin中调用现存的Java代码，并且也能在Java代码中顺利的调用Kotlin代码。这意味着我们可以马上在现有的Java项目中使用上Kotlin，同时所有之前旧的Java也一样有效。

这是很关键，也是我之所以很看好Kotlin的一个原因。

至于怎么相互调用操作，请大家看官方文档关于[Java互操作](http://kotlinlang.org/docs/reference/java-interop.html)的部分。

*这里只说一个方面，关于空安全方面。*

因为Java中的任何应用都可以为null，但是在Kotlin中是默认不可为null的，这使得Kotlin对来自Java的对象要求严格空安全是不现实的。Java声明的类型在Kotlin中会被特别对待，称之为**平台类型**。对这种类型的空检查会放宽，因此他们的安全保证与Java中是相同的。

看下面的例子：
```java
public class Person{
	public String getName(){
		 return null;
	}
}
```

```
val person = Person()
val name = person.name // 编译通过 运行值为null
```
name本是非null变量，因为调用Java对象所以变成平台类型，放宽了类型空检查。

当然，如果你想要延续Kotlin严格空安全机制的话，可是有办法滴。我们需要在编写Java代码时加上@NotNull注解，这个很熟悉吧，在介绍空安全机制的时候说过Kotlin在实现默认非null属性就是这么实现的。然后代码就变成了这样，

```java
public class Person{
	public @NotNull String getName(){
		 return null;
	}
}
```
然后在运行的时候就会报以下的错误：

```
Exception in thread "main" java.lang.IllegalStateException: @NotNull method Person.getName must not return null
	at Person.$$$reportNull$$$0(Person.java)
	at Person.getName(Person.java:9)
	at MainKt.main(main.kt:7)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:147)
```


## 总结
总而言之，经过短短一个月的Kotlin使用，在实际项目中开发展现出的特性应用是让我感到兴奋的。作为一名开发者，在我眼里，Kotlin设计出来不是抛开Java谈的，而是在Java的毛病的基础上，进行的再开发，拥有很多其他语言优秀的特性，同时完全兼容Java。毕竟，对于一家大企业来讲，为了一门新语言完全摒弃一个很成熟的项目进行再开发是不现实的。相反的是，对于项目中Java难于处理的逻辑，Kotlin的优势一览无余，相辅相成，Kotlin和Java配合使用时目前最完美的方案。

但不可否认的是，Kotlin真的让人感到潜力十足，值得大家去试一试。


















