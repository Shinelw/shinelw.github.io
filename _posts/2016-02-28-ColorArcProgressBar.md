---
layout:     post
title:      "ColorArcProgressBar——实现QQ健康步数显示、仪表盘效果"
subtitle:   "ColorArcProgressBar--a beautiful progerssbar"
date:       2016-02-28 15:05:34
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Android组件
---
这是一个可定制的圆形进度条，通过xml参数配置可实现QQ健康中步数的弧形进度显示、仪盘表显示速度、最常见的下载进度条等功能。

Github下载地址：[https://github.com/Shinelw/ColorArcProgressBar](https://github.com/Shinelw/ColorArcProgressBar)

## 效果图
 ![](https://raw.githubusercontent.com/Shinelw/ColorArcProgressBar/master/Demo.gif)
 
# 使用

## 1、在gradle中添加依赖

```gradle
dependencies {
    ...
    compile 'com.github.shinelw.colorarcprogressbar:library:1.0.3'
}
```

## 2、XML

```xml
<com.shinelw.library.ColorArcProgressBar
        android:layout_width="300dp"
        android:layout_height="300dp"
        android:layout_gravity="center_horizontal"
        android:id="@+id/bar1"
        app:is_need_content="true"
        app:front_color1="@color/colorAccent"
        app:max_value="100"
        app:back_width="10dp"
        app:front_width="10dp"
        app:total_engle="360"
        app:is_need_unit="true"
        app:string_unit="百分比%"
        app:back_color="@android:color/darker_gray"
        android:layout_marginBottom="150dp"
        />
```

## 3、代码

```java
progressbar.setCurrentValues(100);
```

## 4、自定义

### 1）定义圆弧度数

```xml
 app:total_engle="270"  
```

### 2）定义渐变色

```xml
app:front_color1="#00ff00"
app:front_color2="#ffff00"
app:front_color3="#ff0000"
```

### 3)定义两条圆弧的粗细

```xml
app:back_width="2dp"
app:front_width="10dp"
```

### 4)设置圆弧中显示文字

```xml
app:is_need_unit="true"
app:string_unit="步"
app:is_need_title="true"
app:string_title="截止当前已走"
```

类似QQ健康中当日步数圆弧显示：


![](https://raw.githubusercontent.com/Shinelw/ColorArcProgressBar/master/demo_qq.gif)

### 5）设置圆弧外刻度值

```java
  app:is_need_dial="true"
```


模拟仪表盘：

![](https://raw.githubusercontent.com/Shinelw/ColorArcProgressBar/master/demo_dashboard.gif)




