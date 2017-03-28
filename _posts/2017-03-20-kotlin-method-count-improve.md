---
layout:     post
title:      "Kotlin属性引发的方法数问题"
subtitle:   "Methods several problems induced by Kotlin property"
date:       2017-03-20 16:51:57
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Kotlin
---

# Kotlin属性引发的方法数问题
Kotlin最重要的一个优势就是简洁。说白了就是写的代码比Java来的少，所以很大程度上使用Kotlin开发会减少项目中的代码行数。然而，如果你觉得方法数肯定也减少的话，那就too young too simple了！首先，撇开引入Kotlin标准库的7000个方法数不谈，接下来我们来谈谈Kotlin属性带来的方法数问题。

## 方法数问题

我们知道，与Java不同的是，Kotlin语言中不提供任何方法来定义裸字段。所有的var和var声明都创建的是一个属性。乍一看来，这是一个伟大的优点，它会自动为你生成getter/setter方法。当然，当你在定义属性变量的同时，
它也允许你自行添加一个get或者set方法。

看上去就是下面这样子的：

```
//Kotlin
class Hello {
    var a: Int = 1
    
    var b: Int = 1
        set(value) {value * 3}

}
```
等价于Java中：
```
public final class Hello {
   private int a = 1;
   private int b = 1;

   public final int getA() {
      return this.a;
   }

   public final void setA(int var1) {
      this.a = var1;
   }

   public final int getB() {
      return this.b;
   }

   public final void setB(int value) {
      int var10000 = this.b * 3;
   }
}
```

看上去是不是很简洁，在Kotlin中我们不需要写防御性的getter和setter方法以保证字段private不可见。显然，这让我们在开发过程中少写了很多代码。但这往往需要有代价的，如果你是个Java开发者，对于暴露出来的公开字段，比如常量的处理，代码是长这样的：

```
//Kotlin

class Constants {
    companion object {
        val CONTENT_TYPE = "Content-Type"
        val CONTENT_LENGTH = "Content-Length"
        val CONTENT_ENCODING = "Content-Encoding"
        val POST = "POST"
        val GZIP = "gzip"
        val UNKNOWN_HOST_EXCEPTION = -1
        val SOCKET_TIMEOUT_EXCEPTION = -2
        val CONNECT_TIMEOUT_EXCEPTION = -3
        val SSL_HANDSHAKE_EXCEPTION = -4
        val SSL_EXCEPTION_EXCEPTION = -5
        val CONNECT_EXCEPTION = -6
        val NETWORK_EXCEPTION = -7
        val REQUEST_CANCEL = -8
        ...
        ...
        ...
    }
}
```

在Java代码中等价于：

```java
public final class Constants {
   @NotNull
   private static final String CONTENT_TYPE = "Content-Type";
   @NotNull
   private static final String CONTENT_LENGTH = "Content-Length";
   @NotNull
   private static final String CONTENT_ENCODING = "Content-Encoding";
   @NotNull
   private static final String POST = "POST";
   @NotNull
   private static final String GZIP = "gzip";
   private static final int UNKNOWN_HOST_EXCEPTION = -1;
   private static final int SOCKET_TIMEOUT_EXCEPTION = -2;
   private static final int CONNECT_TIMEOUT_EXCEPTION = -3;
   private static final int SSL_HANDSHAKE_EXCEPTION = -4;
   private static final int SSL_EXCEPTION_EXCEPTION = -5;
   private static final int CONNECT_EXCEPTION = -6;
   private static final int NETWORK_EXCEPTION = -7;
   private static final int REQUEST_CANCEL = -8;
   public static final Constants.Companion Companion = new  Constants.Companion((DefaultConstructorMarker)null);

    public static final class Companion {
      @NotNull
      public final String getCONTENT_TYPE() {
         return Constants.CONTENT_TYPE;
      }

      @NotNull
      public final String getCONTENT_LENGTH() {
         return Constants.CONTENT_LENGTH;
      }

      @NotNull
      public final String getCONTENT_ENCODING() {
         return Constants.CONTENT_ENCODING;
      }

      @NotNull
      public final String getPOST() {
         return Constants.POST;
      }

      @NotNull
      public final String getGZIP() {
         return Constants.GZIP;
      }

      public final int getUNKNOWN_HOST_EXCEPTION() {
         return Constants.UNKNOWN_HOST_EXCEPTION;
      }

      public final int getSOCKET_TIMEOUT_EXCEPTION() {
         return Constants.SOCKET_TIMEOUT_EXCEPTION;
      }

      public final int getCONNECT_TIMEOUT_EXCEPTION() {
         return Constants.CONNECT_TIMEOUT_EXCEPTION;
      }

      public final int getSSL_HANDSHAKE_EXCEPTION() {
         return Constants.SSL_HANDSHAKE_EXCEPTION;
      }

      public final int getSSL_EXCEPTION_EXCEPTION() {
         return Constants.SSL_EXCEPTION_EXCEPTION;
      }

      public final int getCONNECT_EXCEPTION() {
         return Constants.CONNECT_EXCEPTION;
      }

      public final int getNETWORK_EXCEPTION() {
         return Constants.NETWORK_EXCEPTION;
      }

      public final int getREQUEST_CANCEL() {
         return Constants.REQUEST_CANCEL;
      }
      
      private Companion() {
      }
      
      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

发现问题了没有？我只想添加公开字段直接暴露给其他类调用，当然我并不是说防御型调用getter方式不好，但是代价太大！

这里我只是随便列举了十几个字段，就相应的增加了同样数量的方法。

那么，试想一下，我的项目中有1000个公开常量字段，那么我的项目就要相应的增加1000个方法，并且更加糟糕的是，这些方法并不是必要的！

那么，这时候我只想要完成和Java代码中一样的事并且不增加多余的方法数，该怎么办？


## 解决过程

我们到Kotlin编译过程的最后阶段---目标代码生成，定位到PropertyCodegen类，如果生成属性相关代码。

```
public class PropertyCodegen {

    private void gen(
            @Nullable KtProperty declaration, // 属性声明
            @NotNull PropertyDescriptor descriptor,  //描述，包括权限修饰符、注解、类型等。
            @Nullable KtPropertyAccessor getter, // 决定是否生成getter
            @Nullable KtPropertyAccessor setter  //决定是否生成setter
    ) {
        assert kind == OwnerKind.PACKAGE || kind == OwnerKind.IMPLEMENTATION || kind == OwnerKind.DEFAULT_IMPLS
                : "Generating property with a wrong kind (" + kind + "): " + descriptor;
		  //生成注解信息
        genBackingFieldAndAnnotations(declaration, descriptor, false);

		  //根据注解和权限修饰符等信息判断是否自动生成Getter代码
        if (isAccessorNeeded(declaration, descriptor, getter)) {
            generateGetter(declaration, descriptor, getter);
        }
        //根据注解和权限修饰符等信息判断是否自动生成Setter代码
        if (isAccessorNeeded(declaration, descriptor, setter)) {
            generateSetter(declaration, descriptor, setter);
        }
    }
}
```

看代码，我们需要做的是，让**isAccessorNeeded(declaration, descriptor, getter)**和 **isAccessorNeeded(declaration, descriptor, setter)**的判断都返回false，不让其执行到**generateGetter(declaration, descriptor, getter)**和**generateSetter(declaration, descriptor, setter)**代码，阻止生成属性的setter/getter方法。

我们看看**isAccessorNeeded**里做了什么事。

```
  private boolean isAccessorNeeded(
            @Nullable KtProperty declaration,
            @NotNull PropertyDescriptor descriptor,
            @Nullable KtPropertyAccessor accessor
    ) {
    	//如果是Const修饰或者使用@JvmFiled注解的，直接返回false
        if (isConstOrHasJvmFieldAnnotation(descriptor)) return false;

        boolean isDefaultAccessor = accessor == null || !accessor.hasBody();

        //不要为DefaultImpls中的默认访问器生成接口属性的访问器
        if (kind == OwnerKind.DEFAULT_IMPLS && isDefaultAccessor) return false;
		
        if (declaration == null) return true;

        //委托或扩展属性只能通过访问器引用
        if (declaration.hasDelegate() || declaration.getReceiverTypeReference() != null) return true;

        //伴随对象属性始终应具有访问器，因为它们的后备字段被移动/复制到外部类
        if (isCompanionObject(descriptor.getContainingDeclaration())) return true;

        //来自多个类的非常量属性具有访问者，而不管可见性
        if (isNonConstTopLevelPropertyInMultifileClass(declaration, descriptor)) return true;

        //私有类属性仅在这些访问者不平凡的情况下才具有访问者
        if (Visibilities.isPrivate(descriptor.getVisibility())) {
            return !isDefaultAccessor;
        }

        return true;
    }
```

从代码我们可以看出，有三种情况会返回false，从而不生成getter/setter代码。我们来看这三种情况的实现以及功能验证。

### 情况一：const修饰或者@JvmFiled注解

```
public static boolean isConstOrHasJvmFieldAnnotation(@NotNull PropertyDescriptor propertyDescriptor) {
        return propertyDescriptor.isConst() || hasJvmFieldAnnotation(propertyDescriptor);
    }
```

可以看到，当属性描述的_isConst()_为TRUE时，即对应以下情况：

```
const val a = 1。
```

再看看注解，

```
fun DeclarationDescriptor.hasJvmFieldAnnotation(): Boolean {
    return findJvmFieldAnnotation() != null
}

fun DeclarationDescriptor.findJvmFieldAnnotation() = 
	DescriptorUtils.getAnnotationByFqName(annotations, FqName("kotlin.jvm.JvmField"))
```

当定义的属性找到了@JvmField时，返回TRUE,即对应以下情况：

```
@JvmField var a
```

接下来，我们来验证我们的想法。

```
class Hello {
    companion object {
        const val A = 1
        @JvmField val B = 2
    }
    
}
```

转换为Java代码，

```
public final class Hello {
   public static final int A = 1;
   @JvmField
   public static final int B = 2;
   public static final Hello.Companion Companion = new Hello.Companion((DefaultConstructorMarker)null);

   public static final class Companion {
      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

果然，A和B均没有生成响应的getter/setter方法。


### 情况二：当属性类型为Val

可想而知，当属性类型为val时，属性值则不可改变，所以只会生成getter访问方法，而不会生成setter方法。

```
boolean isDefaultAccessor = accessor == null || !accessor.hasBody();
 
if (kind == OwnerKind.DEFAULT_IMPLS && isDefaultAccessor) {
	return false;
}
```

当属性声明为val时，访问器accessor，也就是**property.getSetter()**返回null。

```

 public KtPropertyAccessor getSetter() {
     for (KtPropertyAccessor accessor : getAccessors()) {
         if (accessor.isSetter()) return accessor;
     }

     return null;
 }
    
 public boolean isSetter() {
        KotlinPropertyAccessorStub stub = getStub();
        if (stub != null) {
            return !stub.isGetter();
        }
        return findChildByType(KtTokens.SET_KEYWORD) != null;
    }
```

以下验证此种情况。

```
//Kotlin
class Hello {
	val a: Int = 1
}
```

在Java中等价转换为：

```
public final class Hello {
   private final int a = 1;

   public final int getA() {
      return this.a;
   }
}
```


### 情况三： 权限访问设置为private

```
 boolean isDefaultAccessor = accessor == null || !accessor.hasBody();
 if (Visibilities.isPrivate(descriptor.getVisibility())) {
            return !isDefaultAccessor;
        }
```
从代码可以看出，具体分为两种情形：
1. 当属性的访问权限为**Visibilities.isPrivate**时，并且是默认访问器时，不自动生成访问器。

以下验证此种情况。

```
//Kotlin

class Hello {
   private var a : Int = 1
}
```

在Java中转换为：

```
public final class Hello {
   private int a = 1;
}
```

2. 当属性的访问权限为**Visibilities.isPrivate**时，但不是默认访问器时，同样会自动生成访问器。

以下验证此种情况。

```
class Hello {

    private var a: Int
        get() = a
        set(value) {value+1}
}
```

在Java中等价转换为：

```
public final class Hello {
   private final int getA() {
      return this.getA();
   }

   private final void setA(int value) {
      int var10000 = value + 1;
   }
}
```
我们会发现很神奇的现象，等价成Java代码后，a字段不见了，但是生成了private访问权限的getter/setter。

## 总结

通过分析Kotlin编译过程中目标代码生成的阶段，我们发现了几种不自动生成getter/setter访问器代码的情形。但是，对于一开始应用的场景，只有情况一最为合适。即：

当有大量公共字段需要暴露给外界调用时，我们可以通过添加const修饰符或者@JvmField注解的方式进行。

所以，一开始的代码就可以转变成一下两种情形：

```
//Kotlin

class Constants {
    companion object {
        const val CONTENT_TYPE = "Content-Type"
        const val CONTENT_LENGTH = "Content-Length"
        const val CONTENT_ENCODING = "Content-Encoding"
        const val POST = "POST"
        const val GZIP = "gzip"
        const val UNKNOWN_HOST_EXCEPTION = -1
        const val SOCKET_TIMEOUT_EXCEPTION = -2
        const val CONNECT_TIMEOUT_EXCEPTION = -3
        const val SSL_HANDSHAKE_EXCEPTION = -4
        const val SSL_EXCEPTION_EXCEPTION = -5
        const val CONNECT_EXCEPTION = -6
        const val NETWORK_EXCEPTION = -7
        const val REQUEST_CANCEL = -8
        ...
        ...
        ...
    }
}
```

或者是：

```
class Constants {
    companion object {
        @JvmField val CONTENT_TYPE = "Content-Type"
        @JvmField val CONTENT_LENGTH = "Content-Length"
        @JvmField val CONTENT_ENCODING = "Content-Encoding"
        @JvmField val POST = "POST"
        @JvmField val GZIP = "gzip"
        @JvmField val UNKNOWN_HOST_EXCEPTION = -1
        @JvmField val SOCKET_TIMEOUT_EXCEPTION = -2
        @JvmField val CONNECT_TIMEOUT_EXCEPTION = -3
        @JvmField val SSL_HANDSHAKE_EXCEPTION = -4
        @JvmField val SSL_EXCEPTION_EXCEPTION = -5
        @JvmField val CONNECT_EXCEPTION = -6
        @JvmField val NETWORK_EXCEPTION = -7
        @JvmField val REQUEST_CANCEL = -8
        ...
        ...
        ...
    }
}

```

在暴露给外界调用的同时，不会新增多余的方法数。









