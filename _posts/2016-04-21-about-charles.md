---
layout:     post
title:      "Charles:移动端设备网络抓包"
subtitle:   "Charles:An application to catch the data using mobile"
date:       2016-04-21 15:45:03
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 工具
    - 抓包
---

今天安利一款软件，Charles。最近在研究应用安全的东西，经常需要对应用访问网络进行抓包，然后发现Charles这款软件，最突出的特点就是简单好用易上手~啊哈哈哈
<!--more-->

首先是应用界面，如下：
![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/charles/Screen%20Shot%202016-04-21%20at%203.14.04%20PM.png)

下载地址：<http://www.charlesproxy.com>

下载下来以后会提示只有30天免费试用期，这时候只有两种选择，要么购买，要么破解。（对于我这种穷学生来说，只好破解了= =）
目前最新版本是3.11.4，所以google搜索一下charles 3.11.4 注册文件，要是你还懒得搜索的话，可以[点这里](https://github.com/Shinelw/Android/raw/master/BlogPicture/charles/charles.jar)，我已经把注册文件保存在github上，如需自取~
至于安装的话，鉴于我的环境是mac osx，所以就讲一下mac中的破解方法。

### **破解方法：**
1. 打开Applications找到Charles软件，显示包内容，打开Java文件夹。
2. 把下载好的charles.jar文件替换到Java文件夹中，重启应用就ok了。

接下来是配置方法，在3.10版本以后，配置方法就变得特别简单，只要跟随Help中SLL Proxying中的操作就ok了。
![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/charles/Screen%20Shot%202016-04-21%20at%203.47.52%20PM.png)


### **配置**
1. 安装本地证书。
	点击Help中SLL Proxying中的Install Charles Root Certificate，然后输入密码，选择全部信任。
2. 移动设备网络配置。
	点击Help中SLL Proxying中 Install Charles Root Certificate on a Mobile Device or Remote Browser…,然后就会跳出如下弹窗。
	![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/charles/Screen%20Shot%202016-04-21%20at%203.47.17%20PM.png)
	
	根据弹窗信息更改配置，如下图： 
	![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/charles/Screen%20Shot%202016-04-21%20at%203.51.36%20PM.png)
	
3. 完成配置以后，手机浏览器打开 <http://www.charlesproxy.com/getssl> ，就会下载ssl证书，并且进行安装。如下图，当安装完成以后右键想要抓包的网址选择Enable SSL Proxying就可以对HTTPS数据进行抓包了！
![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/charles/Screen%20Shot%202016-04-21%20at%203.53.23%20PM.png)

抓包效果图：

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/charles/Screen%20Shot%202016-04-21%20at%203.50.53%20PM.png)

以上，就是所有的配置过程。

### **总结**
抓包对于程序员来说应该是一个必备的技能点，而且很有趣。比如在抓包的过程中会发现很多应用对于密码都是明文的= =
继续加油吧~
