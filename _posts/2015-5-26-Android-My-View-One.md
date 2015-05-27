---
layout: post
title: "Android自定义View（一）"
date: 2015-5-26
category: Android
tag: 自定义View
---
####基础知识

> 引用官方文档一段话觉得说的挺好

设计良好的类总是相似的。它使用一个好用的接口来封装一个特定的功能，它有效的使用CPU与内存，等等。为了成为一个设计良好的类，自定义的view应该:

- 遵守Android标准规则。
- 提供自定义的风格属性值并能够被Android XML Layout所识别。
- 发出可访问的事件。
- 能够兼容Android的不同平台。

首先Android平台上面的自定义View需要满足以上四种条件，自定义View说白了就是Android系统自带的控件满足不了我们的需求，比如说我们现在来定义一个简单的控件

![图片](/img/2015-5-26/sample.png)

就这样一个图片我的思路是用一个Framelayout一个底部图片，上面放一个Linearlayout对齐底部，中间显示文字，然后给这个控件设置一个透明颜色，很简单的一个小控件。

- 实现自定义底部图片
- 实现自定义下部文字

####继承一个View
Android中所有自定义View都需要继承View，比如说：

```java
public class MyView extends View {

    public MyView(Context context) {
        super(context);
    }

    public MyView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MyView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
//
//    public MyView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
//        super(context, attrs, defStyleAttr, defStyleRes);
//    }
}
```

前面三个constructor是要求最低API7，最后那个我注释掉的是Android L新出的 0.0

Context：传入的上下文

####实现自定义属性

AttributeSet：可以通过传入一个attr达到在XML中填写我们自己的属性，例如

```xml
 <com.smallsoho.MyImage
            android:layout_width="fill_parent"
			android:layout_height="fill_parent"
            android:layout_weight="1"
            myapp:text="逗比"
			myapp:drawable="@drawable/ic_luncher"/>
```

稍后我会讲解这个这个详细的使用方法。

####其余两个属性

**defStyleAttr**：这个是当前Theme中的一个attribute，是指向style的一个引用，当在layout xml中和style中都没有为View指定属性时，会从Theme中这个attribute指向的Style中查找相应的属性值，这就是defStyle的意思，如果没有指定属性值，就用这个值，所以是默认值，但这个attribute要在Theme中指定，且是指向一个Style的引用，如果这个参数传入0表示不向Theme中搜索默认值

**defStyleRes**：这个也是指向一个Style的资源ID，但是仅在defStyleAttr为0或defStyleAttr不为0但Theme中没有为defStyleAttr属性赋值时起作用
