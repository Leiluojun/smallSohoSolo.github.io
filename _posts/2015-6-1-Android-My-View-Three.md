---
layout: post
title: "Android自定义View（三）"
date: 2015-6-1
category: Android
tag: 自定义View
---

####前言
上一篇博文我们继承了Linearlayout来达到简单的组合控件，那么这篇讲解一下真正的继承ViewGroup和继承View的完全自定义View使用方法。
这跟三个函数有关，需要我们重写onDraw(),onMeasure(),onLayout()三个函数在类中，以达到我们的目的，首先我们先做试图上面的一个自定义，在下一篇中我们将会讲解对手势的控制，有木有好激动！～！～～！

####流程和三个函数的功能

Android绘制View会按照一个流程来做

**onMeasure() --> onLayout --> onDraw()**

- View本身大小多少，这由onMeasure()决定
- View在ViewGroup中的位置如何，这由onLayout()决定
- 绘制View，onDraw()定义了如何绘制这个View。

接下来按照这个流程进行讲解

####onMeasure()
> 引用郭霖大神的一段话（我会修改的更加通俗易懂一点） [http://blog.csdn.net/guolin_blog/article/details/16330267](http://blog.csdn.net/guolin_blog/article/details/16330267)
> ps：大神博客很好，推荐大家看看

measure是测量的意思，那么onMeasure()方法顾名思义就是用于测量视图的大小的。View系统的绘制流程会从ViewRoot的performTraversals()方法中开始，在其内部调用View的measure()方法。measure()方法接收两个参数，widthMeasureSpec和heightMeasureSpec，这两个值分别用于确定视图的宽度和高度的规格和大小。

MeasureSpec的值由specSize和specMode共同组成的，其中specSize记录的是大小，specMode记录的是规格。specMode一共有三种类型，如下所示：

**1.EXACTLY**

表示父视图希望子视图的大小应该是由specSize的值来决定的，系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

**2.AT_MOST**

表示子视图最多只能是specSize中指定的大小，开发人员应该尽可能小得去设置这个视图，并且保证不会超过specSize。系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

**3.UNSPECIFIED**

表示开发人员可以将视图按照自己的意愿设置成任意的大小，没有任何限制。这种情况比较少见，不太会用到。

####onLayout()


####onDraw()


####总结
