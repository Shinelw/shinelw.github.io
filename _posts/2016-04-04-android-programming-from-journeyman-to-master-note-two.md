---
layout:     post
title:      "《Android开发进阶 从小工到专家》读书笔记 2"
subtitle:   "《Android Programming from journeyman to master》 Note II"
date:       2016-04-04 22:50:22
author:     "Shinelw"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 读书笔记
---

# 第二章 创造出丰富多彩的UI View与动画

## 1. 重要的View控件：ListView 与 RecyclerView

### **ListView**

列表数据的显示需要4个元素，分别为：

1. 用来显示列表的ListView
2. 用来把数据映射到ListView上的Adapter
3. 需要展示的数据集
4. 数据显示的View模板

其中重点是Adapter的实现，需要实现的函数为：

1. getCount():获取数据的个数
2. getItem(int): 获取position数据的位置
3. getItemId(int): 获取position位置的数据id，一般直接返回position即可
4. getView(int,View,ViewGroup): 获取position位置上的ItemView视图


**ListView的ItemView复用机制：**

```java
	public View getView(int position, View convertView, ViewGroup parent){
		View view = null;
		//视图缓存
		if(convertView != null){
			view = convertView;
		}else{
			//重新加载视图
		}
		//进行数据绑定
		//返回Item View
		return view;
	}
```

其中convertView代表缓存的Item View，如果有缓存的话convertView不为空，此时直接复用该视图；否则需要重新创建一个新的视图，最后绑定数据并且将该Item View返回。

**ListVie数据更新：**

ListView运用了Adapter模式，在Adapter类中运用了观察者模式实现数据源发生改变后的更新。

大致流程如下：

1. setAdapter时创建AdapterDataSetObserver对象，注册到mAdapter中；
2. 在数据源发生改变时调用adapter的notifyDataSetChanged()；
3. 在notifyDataSetChanged()方法中调用DataSetObservable对象的notifyChanged方法，在该方法中通知所有观察者数据发生了变化，使观察者进行相关的操作；
4. notifyChanged中调用观察者的onChanged()方法，在onChanged()方法中调用ViewGroup的requestLayout()进行重新策略布局、绘制整个ListView的Item View.

### **RecyclerView**

RecyclerView封装了ViewHolder类型，该类型中有一个itemView，代表的就是每一项数据的根视图，需要在构造函数中传递给ViewHolder对象。

实现RecyclerView的Adapter需要实现一下两个函数：

1. onCreateViewHolder():创建ViewHolder
2. onBindViewHolder():绑定数据

RecyclerView优势（扩展性强）：

1. 使用ViewHolder，对ListView的Adapter进行了再次封装，不需要考虑缓存问题。
2. 将布局方式抽象为LayoutManager，默认提供了LinearLayoutManager、GridLayoutManager、StaggeredGridLayoutManager 3中布局，对应为线性布局、网格布局、交错网格布局。
3. 对于ItemView的控制更精细，可以通过ItemDecoration为Item View添加装饰，手动设置分割线等等；用ItemAnimator为Item View添加动画效果。

## 2. 自定义控件

自定义控件重要的知识点：

- View的测量与布局
- View的绘制
- 处理触摸事件
- 动画效果

### **自定义View**

继承View实现自定义View，过程如下：

1. 继承自View创建自定义控件；
2. 如有需要自定义View属性，也就是在values/attrs.xml中定义属性集；
3. 在xml中引入命名控件，设置属性；
4. 在代码中读取xml中的属性，初始化视图；
5. 测量视图大小；
6. 绘制视图内容。

### **View的尺寸测量**

measure()方法接收两个参数：

- widthMeasureSpec
- heightMeasureSpec
这两个值分别用于确定视图的宽度、高度的规格和大小。

MeasureSpec的值由：

- specSize: 记录大小
- specMode: 记录规格

共同组成。

**specMode的三种类型：**

1. **EXACTLY**: 子视图大小由specSize的值决定，对应match_parent、具体的数值。
2. **AT_MOST**: 子视图最多只能specSize的大小， 对应warp_content。
3. **UNSPECIFIED**: 开发人员将视图按照自己的意愿设置成任意的大小，没有任何限制。（基本没有用到）

### **Canvas与Paint(画布与画笔)**

整个View就是一张画布，也就是Canvas。通过画笔Paint在这张画布上绘制各种各样的图形、元素，修改画笔的属性可以将同一个元素绘制出不同的效果。

**Canvas中save()与restore()**

- **save()**: 存储当前矩阵和剪裁状态到一个私有的栈中。随后调用translate，scale，rotate，skew，concat，clipRect，clipPath等函数还是会正常执行，但是调用了restore()之后，这些调用产生的效果就会失效，在save之前的Canvas状态都会被恢复。
- **restore()**: 恢复save之前的状态。


###  **自定义ViewGroup**

应用场景：当我们需要自定义子视图的排列方式时。






