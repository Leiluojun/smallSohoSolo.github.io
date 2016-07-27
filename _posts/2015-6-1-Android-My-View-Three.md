---
layout: post
title: "Android自定义View（三）"
date: 2015-5-28
category: Android
tag: 自定义View
---

####前言

本篇博客将上个博客进行深入的一点研究，包括对布局的初步探讨和对view的绘制进行一些探讨。

####实现自定义View的绘制

首先我们需要重写自定义view的onDraw()方法，重写此方法可以对控件如何绘制进行控制，

```java
public class MyViewTwo extends View{

    public MyViewTwo(Context context) {
        super(context);
    }

    public MyViewTwo(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MyViewTwo(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    //
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
    }
}
```

onDraw()会给我们一个canvas参数，我们绘制的东西需要绘制在这个canvas中，这里注意，onDraw方法会在重绘的时候被频繁调用，所以初始化操作最好放在其他的参数中，如果在onDraw方法中频繁调用，那么会产生昂贵的代价，造成内存抖动。并且使UI变得卡顿。
我们需要创建一个Paint画笔，在canvas中进行绘画

- 绘制什么，由canvas制定
- 如何绘制（也就是绘制的各种参数），由Paint制定

