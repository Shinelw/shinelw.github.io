---
layout:     post
title:      "Ubuntu 15.10 配置Android Studio出现的问题"
subtitle:   "Ubuntu 15.10 Install Android Studio Trouble"
date:       2016-04-08 15:25:06
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - AndroidStudio
    - Ubuntu
---

最近重新安装了Ubuntu系统，升级到15.10，在一番折腾重新配置Android Studio的时候，发现不能正确关联本地的Android SDK。

显示错误如下：

>unable to run mksdcard sdk tool

google了一番发现是升级到15.10系统以后系统内缺少32位的lib。

所以，安装上相应的库就可以了。

执行以下命令：

```bash
sudo apt-get install lib32z1 lib32ncurses5  lib32stdc++6
```

完美解决~
