---
layout: post
title: "Android Handler的一点思考"
date: 2015-7-20
category: Android
tag: Handler
---

##Handler的一点思考

####什么是handler

handler是Android给我们提供的一种消息接受机制，通过handler可以实现在子线程中更新UI，默认生成的handler是运行在ui线程中的。

####什么是Looper和MessageQueue

handler中存在一个MessageQueue（消息队列）
Looper是一只“手”，这只手无限循环MessageQueue从中拿出来要处理的message，然后交给handler处理，默认生成的handler中的looper是主线程也就是ui线程的looper，所以默认生成的handler是可以处理更新ui的操作的。

```java
handler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                switch (msg.what) {
                    case 1:
                        textView.setText("button被点击");
                        Log.d("chenlongbo", "操作1完成");
                        break;
                    case 2:
                        Log.d("chenlongbo", "操作2完成");
                        break;
                    default:
                        break;
                }
            }
        };
findViewById(R.id.button).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                handler.sendEmptyMessage(1);
            }
        });

```

上述代码实现了一个运行在主线程中的handler实现的方法。
Looper是隐含在其中的
这里的handler是直接new出来的，所以其中的handler是ui线程的looper，还有一种方式是利用handlerthread来达到使用子线程的looper处理handler操作

####什么是HandlerThread

HandlerThread是Thread的子类，其中自带一个Looper和MessageQueue，用来达到按次处理消息的功能，
我们知道不可以在主线程中做耗时操作，如果直接在默认的handler的handlermessge方法中Thread.sleep（5000） 会使卡顿UI线程
HandlerThread给我们提供了一个可以在handler中处理耗时操作的可能。
```java
        HandlerThread handlerThread = new HandlerThread("longlongtest");
        handlerThread.start();
		//此处必须start，不然无法生成子线程的looper
        handler = new Handler(handlerThread.getLooper()) {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                switch (msg.what) {
                    case 1:
                        try {
                            Thread.sleep(5000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        Log.d("chenlongbo", "操作1完成");
                        break;
                    case 2:
                        Log.d("chenlongbo", "操作2完成");
                        break;
                    default:
                        break;
                }

            }
        };
        findViewById(R.id.button).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                handler.sendEmptyMessage(1);
            }
        });

```


此时执行耗时操作不会卡顿ui线程，因为我们是使用子线程的looper并且在子线程中做耗时操作，那么就不会卡顿。
注意，因为此时handler处于子线程中，Android规定子线程无法操作ui。所以此处无法操作ui

####HandlerThread给我们解决了什么问题

在Android的多线程操作中，如果两个子线程的处理有交集，也就是简单的生产者交易者问题，第一个线程不完成，第二个线程没有第一个线程的结果无法继续执行下去，那么此时我们使用HandlerThread就可以实现了

假设A代表一个耗时操作，B代表另外一个耗时操作   B需要获取A的结果来继续执行

```java

        HandlerThread handlerThread = new HandlerThread("longlongtest");
        handlerThread.start();

        handler = new Handler(handlerThread.getLooper()) {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                switch (msg.what) {
                    case 1:
                        try {
                            Thread.sleep(5000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        Log.d("chenlongbo", "操作1完成");
                        break;
                    case 2:
                        try {
                            Thread.sleep(5000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        Log.d("chenlongbo", "操作2完成");
                        break;
                    default:
                        break;
                }

            }
        };
        handler.sendEmptyMessage(1);
        handler.sendEmptyMessage(2);

```

####总结

Handler给我们提供了一套消息机制，可以让我们顺次执行我们放在队列里面的操作，也可以在子线程中执行耗时操作之后发送消息到主线程的messagequeue中之后更新主线程，但是在handler的handlemenssage方法中是不能执行耗时操作的，那会卡顿ui线程

HandlerThread是对handler的扩展使用，可以让我们使用handler的消息机制来达到子线程中的耗时操作进行一种交互，在队列中顺次执行
我们发送进队列的消息。具体的使用十分的灵活