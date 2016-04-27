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

通常情况下，在开发过程中打包apk是一件很简单的事，主要可以通过两种方式：一种是用Eclipse或者Android Studio集成开发环境直接生成apk；另一种是使用Ant工具在命令行方式下打包APK。不过不管哪种方式，打包APK的本质过程都是一样的。

APK文件其实就是一个压缩包，当你解压以后会发现它内部主要就是资源文件和classes.dex。这个classes.dex就是Android系统Dalvik虚拟机的可执行文件。

Android Developer官网对APK打包有具体的介绍，[点击这里查看](http://developer.android.com/intl/zh-cn/sdk/installing/studio-build.html)。

同时提供一张APK打包的流程图，如下：

![](https://raw.githubusercontent.com/Shinelw/shinelw.github.io/assets/build.png)


## 具体步骤 

