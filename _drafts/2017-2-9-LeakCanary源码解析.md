---
layout: post
title: "LeakCanary源码解析"
date: 2017-2-9
category: Android
tag: optimization
---

LeakCanary是Square公司开源的内存泄漏检测工具，可以帮助我们在开发中弹出内存泄漏的提示

![screenshot](/Users/longlong/Desktop/leakcanary.png)

仓库地址: [https://github.com/square/leakcanary](https://github.com/square/leakcanary)

### 使用方法###

在你的build.gradle文件中写入

```gr
dependencies {
   debugCompile 'com.squareup.leakcanary:leakcanary-android:1.5'
   releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
   testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
}
```

在Application中写入

```java
public class ExampleApplication extends Application {
  @Override 
  public void onCreate() {
    super.onCreate();
    if (LeakCanary.isInAnalyzerProcess(this)) {
      // This process is dedicated to LeakCanary for heap analysis.
      // You should not init your app in this process.
      return;
    }
    LeakCanary.install(this);
    // Normal app init code...
  }
}
```

之后LeakCanary就会在你的app有内存泄漏的时候弹出上图的notification

使用一个库的方法大多都十分简单，但是公共的库都有自己的局限性，并不能完全的匹配我们的业务，这也是为什么图片加载库以及各种Hotfix方案百家争鸣，符合自己需求的库才是最好的库，所以懂得一个仓库的原理十分重要，我们就可以定制自己的leakcanary，下面就来进行一次深入的解析LeakCanary。

### 整体结构###

首先Clone下来源码我们会发现LeakCanary的结构

![file_struct](/Users/longlong/Desktop/file_struct.png)

- analyzer
- android
- android-no-op
- sample: leakcanary的demo
- watcher

### 原理概括###

### 逐步分析源码###