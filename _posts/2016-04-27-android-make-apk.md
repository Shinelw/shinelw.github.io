---
layout:     post
title:      "Android APK打包流程"
subtitle:   "Android APK Build Process"
date:       2016-04-27 16:52:56
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Android
---

## 概述

今天主要讲一下Android程序的生成步骤，即Android打包成APK的流程。

通常情况下，在开发过程中打包APK是一件很简单的事，主要可以通过两种方式：一种是用Eclipse或者Android Studio集成开发环境直接生成APK；另一种是使用Ant工具在命令行方式下打包APK。不过不管哪种方式，打包APK的本质过程都是一样的。

APK文件其实就是一个压缩包，当你解压以后会发现它内部主要就是资源文件和classes.dex。这个classes.dex就是Android系统Dalvik虚拟机的可执行文件。

Android Developer官网对APK打包有具体的介绍，[点击这里查看](http://developer.android.com/intl/zh-cn/sdk/installing/studio-build.html)。

同时提供一张APK打包的流程图，如下：

![](https://raw.githubusercontent.com/Shinelw/shinelw.github.io/master/assets/build.png)


## 打包流程

### 1. 打包资源文件，生成R.java文件

打包资源的工具是**aapt（The Android Asset Packaing Tool）**，位于android-sdk/platform-tools目录下。在这个过程中，项目中的AndroidManifest.xml文件和布局文件XML都会编译，然后生成相应的R.jva。

### 2. 处理aidl文件，生成相应的Java文件

这一过程中使用到的工具是**aidl（Android Interface Definition Language）**，即Android接口描述语言。位于android-sdk/platform-tools目录下。aidl工具解析接口定义文件然后生成相应的Java代码接口供程序调用。

如果在项目没有使用到aidl文件，则可以跳过这一步。

### 3. 编译项目源代码，生成class文件

项目中所有的Java代码，包括*R.java*和*.aidl*文件，都会变Java编译器（**javac**）编译成*.class*文件，生成的class文件位于工程中的bin/classes目录下。

### 4. 转换所有的class文件，生成classes.dex文件

**dx**工具生成可供Android系统Dalvik虚拟机执行的classes.dex文件，该工具位于android-sdk/platform-tools 目录下。

任何第三方的*libraries*和*.class*文件都会被转换成*.dex*文件。

dx工具的主要工作是将Java字节码转成成Dalvik字节码、压缩常量池、消除冗余信息等。

### 5. 打包生成APK文件

所有没有编译的资源（如images等）、编译过的资源和*.dex*文件都会被**apkbuilder**工具打包到最终的*.apk*文件中。

打包的工具**apkbuilder**位于 android-sdk/tools目录下。apkbuilder为一个脚本文件，实际调用的是android-sdk/tools/lib/sdklib.jar文件中的**com.android.sdklib.build.ApkbuilderMain**类。

### 6. 对APK文件进行签名

一旦APK文件生成，它必须被签名才能被安装在设备上。

在开发过程中，主要用到的就是两种签名的**keystore**。一种是用于调试的debug.keystore，它主要用于调试，在Eclipse或者Android Studio中直接run以后跑在手机上的就是使用的debug.keystore。另一种就是用于发布正式版本的keystore。

### 7. 对签名后的APK文件进行对齐处理

如果你发布的apk是正式版的话，就必须对APK进行对齐处理，用到的工具是**zipalign**，它位于android-sdk/tools目录下。

对齐的主要过程是将APK包中所有的资源文件距离文件起始偏移为4字节整数倍，这样通过内存映射访问apk文件时的速度会更快。

对齐的作用就是减少运行时内存的使用。

## Note

APP应用中方法数都被限制为64K。如果你的应用中方法数达到这个上限，在打包过程中就会输出以下的错误信息：

>Unable to execute dex: method ID not in [0, 0xffff]: 65536.

至于为什么会限制64k的上限，接下来我会分析。当然，官方开发文档也有分析，[可以看这里](http://developer.android.com/intl/zh-cn/tools/building/multidex.html)。



