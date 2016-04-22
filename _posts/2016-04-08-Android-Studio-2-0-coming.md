---
layout:     post
title:      "Android Studio 2.0 新特性体验"
subtitle:   "Android Studio 2.0 is coming"
date:       2016-04-08 20:46:06
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - AndroidStdio
---

今天微博被各种 Android Studio 2.0 刷屏了，赶紧更新体验了一把。

>这里推荐一个博客[Android Developers Blog](http://android-developers.blogspot.com)，这是Google官方博客，Android的最新技术都会火热更新在上面。

接下来的新特性介绍内容大致翻译自 Android Developers Blog 文章 [Android Studio 2.0](http://android-developers.blogspot.com/2016/04/android-studio-2-0.html)。

Android Studio 2.0 包括以下可供开发者用于工作流程的新特性：

- **Instant Run(即时运行)** - 面向每个热爱快速构建应用的开发者。对应用做一些修改，就可以显示在运行的app上。对于多次编译/运行调试的应用来说，Instant Run（即时运行）会节省很多时间。
- **Android Emulator(安卓虚拟器)** - 新的模拟器大约会比之前版本的虚拟器快3倍，通过ADB增强你可以更加快速的把数据和应用放入虚拟器中，比真机要快上10倍。跟真机一样，官方的虚拟器也包括了 Google Play Services（谷歌服务框架），所以可以在虚拟器上测试更多的API功能。最后，新的模拟器拥有更多的特性，比如管理电话，电池，网络，GPS，等等。
- **Cloud Test Lab Integration(云服务集成)** - 一旦编写好，在任何地方都可以运行。利用继承在 Android Studio 内部的 Cloud Test Lab 云服务，可以更加快速、方便地在更广泛的 Android 真机上测试。
- **App Indexing Code Generation & Test** - 通过 Android Studio 中的 App Indexing 可以在你的应用中添加URL来提升你的应用在 Google Search 中的可见度。
- **GPU Debugger Preview(GPU调试预览)** - 对于那些开发基于 OpenGL ES 的游戏或应用来说，现在可以看见每一帧和GL的状态。
- **IntelliJ 15 Update** - Android Studio 基于IntelliJ 构建，已升级到最新版本。

### **深入了解新特性**

**Instant Run**

当你点击Instant Run 按钮时，Instant Run 会分析你这次所做的更改然后决定怎么以更快的方式部署运行新的代码。

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Android-Studio-2.0-coming/image02.png)

需要注意的是，Instant Run 需要运行在API 14或高于14的Android 设备或虚拟器上。

**Android Emulator**

新的Android 虚拟器在CPU、RAM和IO上都比之前快了3倍以上。当你准备编译时，ADB增强让push速度快10倍！在绝大多数情况下，在官方虚拟器上开发会快于真机，并且想Instant Run 这样的新特性在新的虚拟器在能更好的工作。

除此之外，新的Android 虚拟器拥有一个全新的界面和传感器控制面板，并且你可以直接将APK拉入虚拟器中完成快速安装，对窗口进行扩大缩放，使用多种触控方法等等。

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Android-Studio-2.0-coming/image00.png)

**Cloud Test Lab**

Cloud Test Lab 允许你在更广泛的设备和虚拟器上测试你的应用。一旦你完成了初始测试，Cloud Test Lab 可以提供给你更大范围的设备来进行测试。即使你没有明确的写了测试，Cloud Test Lab 可以执行基本的测试集以保证你的应用不会Crash。

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Android-Studio-2.0-coming/image06.png)

Android Studio中新的接口允许你配置想要运行在Cloud Test Lab 上的测试文件夹，并且允许你看见测试的结果。

**App Indexing**

现在，通过App Indexing API，你的应用可以在Google搜索中更容易的被用户找到。通过配置AndroidMainfest.xml文件，Android Studio 2.0 可以帮助你创建正确的URL以便运行在Google App Indexing 服务中。当你添加了URL以后，你可以看到以下的效果：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Android-Studio-2.0-coming/image05.png)


**GPU Debugger Preview**

如果你开发OpenGL ES 游戏或者图形应用，现在你拥有一个全新的GPU调试器。即使现在只是一个预览版，但是你可以一帧一帧的识别和调试应用。

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/Android-Studio-2.0-coming/image08.png)


### **Tips**
1. 为了使用Instant Run 和Android虚拟器，对于之前的项目，都需要将gradle 插件版本更新到2.0.0。
2. 之后创建的新项目都默认使用Instant Run。

### **体验感受**
1. Instant Run是版本最大亮点，没有之一。体验下来确实快了很多，而且在你修改界面以后运行后能直接显示该页面效果，而不是重新启动应用。
2. 新的虚拟机感觉噱头大于实际使用效果，还是喜欢用真机测试，爽。
3. 其他特性具体效果在接下来中的开发再慢慢体验啦。


赶紧更新耍起来吧~~~


注：若翻译效果不佳，望请谅解。
