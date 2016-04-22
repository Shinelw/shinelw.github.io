---
layout:     post
title:      "Wish App逆向分析app_device_id字段生成算法"
subtitle:   "Wish App Analysis how to build the app_device_id"
date:       2016-04-21 16:52:56
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Android逆向分析
---

## **概述**

本文对Wish App进行了反编译，对应用中app_device_id字段的生成算法进行了逆向分析。

使用到的工具有：

1. Apktool：获取资源文件和smali 反汇编代码
2. dex2jar：反编译apk，将其中的classes.dex转化成jar文件
3. jd-gui：打开jar文件，查看java源码

反编译环境：mac osx


## **apk反编译获得java代码和smali反汇编代码**

**1. 从google play下载Wish.apk**

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%206.24.54%20PM.png)

**2. 使用dex2jar把apk解包，将其中的classes.dex转换为jar文件**

打开终端，执行命令： **d2j-dex2jar Wish.apk** ，如下图，得到Wish-dex2jar.jar

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%206.33.32%20PM.png)

**3. 使用jd-gui查看Wish-dex2jar.jar文件，即java源代码**

在jd-gui中打开Wish-dex2jar.jar文件，得到Java的源代码，如下图：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%206.36.44%20PM.png)

**4. 使用Apktool反编译apk，得到smali反汇编代码**

终端执行命令： **apktool d -f Wish.apk -o Wish** , 如下图：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%206.42.38%20PM.png)

此时，得到Wish文件夹，里面包含apk中的xml、切图等资源文件和可供分析的smali反编译代码。

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%206.43.30%20PM.png)


## **分析app_device_id字段生成算法**

**1. 快速定位app_device_id字段代码位置**

jd-gui使用search功能快速定位app_device_id在代码中的位置，如下图：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%206.46.16%20PM.png)

可知，关键代码为：

> paramHttpRequestParams.put("app_device_id", getDeviceId());

应用在每次进行网络请求的时候会上传app_device_id字段，字段对应的值为getDeviceId()方法中返回的值。所以接下来对该方法进行分析。

**2. getDeviceId()方法分析**

我们来先看getDeviceId()的代码实现：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%207.00.37%20PM.png)

从代码中可以很明显的将整个获取DeviceId的过程分为三部分：

（1）从SharedPreferences存储中取出DeviceUuid。

代码段为：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%208.35.24%20PM.png)

（2）如果SharedPreferences中没有保存DeviceUuid值的话，就从本地的文件夹中取，文件地址具体地址为/Document/Wish/device_data_。若得到值以后，通过SharedPreferences保存起来。

代码段为：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%208.36.23%20PM.png)

（3）如果本地文件中也没有的话，就创建一个（此时的情况跟安装应用后第一次进入的情形一样）。创建的过程使用了随机产生的方式，即java.util.UUID类中提供的randomUUID()方法。格式为：23c2add6-aa30-4442-97c8-81930766f089。 然后将得到的值以SharedPreferences的方式保存起来。

代码段为：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%208.37.22%20PM.png)

当获取到随机值后，新开线程将值保存在本地固定文件夹/Docment/Wish/device_data_下。

代码段为：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%208.38.05%20PM.png)

由于在run方法中出现/*Error*/，run()方法中的代码无法进行反编译。所以用smali进行验证。

**3. smali代码分析新开线程保存DeviceUuid值在本地device_data的过程**

smali文件夹中包含一下文件：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%207.30.11%20PM.png)

找到com.contextlogic.wish.api.core中的WishApi.smali，打开该文件定位到getDeviceId方法下，如图：(代码太多，故中间省略。)

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Picture1.png)

逐步定位到新开线程的代码处：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%207.49.17%20PM.png)

从代码中可以看到，定义了一个局部变量writeableDeviceId来保存DeviceUuid值，然后开启一个线程，跳转到WishApi$2.smali中去。

打开WishApi$2.smali代码，

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%207.56.22%20PM.png)

代码中就只有一个run()方法。

在run()方法中，一共就干了两件事，

（1）定位到/Document/Wish/device_data_。如果存在，就直接定位到该文件下；如果不存在，就新建Wish文件夹，新建device_data。

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%208.06.22%20PM.png)

（2）打开FileOutputStream将值写入到device_data_文件中去。

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Picture2.png)


## **总体流程**

大致流程图如下：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-18%20at%208.33.22%20PM.png)

在手机中路径/Docment/Wish/devicedata保存下来的值：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-21%20at%202.10.55%20PM.png)

与抓包得到的数值吻合：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Wish_analysis/Screen%20Shot%202016-04-21%20at%202.09.41%20PM.png)


## **总结：**

由于app_device_id字段对应的值是随机产生的，但是由上述代码分析可知，每一次访问网络的时候都要发送app_device_id字段，这就要求对应值要是唯一的，以确保进行操作的设备是安全的。而这里对实现唯一的方法就是永远只随机产生一次，通过SharedPreferences和File方式分别把值分别保存下来。只有当应用卸载重新安装并且删除本地文件device_data的时候才会重新随机产生一个app_device_id值，然后一般用户是不可以在删除应用的时候同时找到它所保存的文件路径并删除，这就达到了设备ID的唯一性。

