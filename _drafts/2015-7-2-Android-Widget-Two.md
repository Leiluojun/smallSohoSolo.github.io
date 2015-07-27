---
layout: post
title: "Android Widget详解（二）"
date: 2015-7-2
category: Android
tag: Widget
---

####前言

上一篇写了实现一个简单的小控件，这篇进行复杂布局以及交互的讲解。

####可以使用的布局

>官方有话这样说：
A RemoteViews object (and, consequently, an App Widget) can support the following layout classes:

>- FrameLayout
- LinearLayout
- RelativeLayout

>And the following widget classes:

>- AnalogClock
- Button
- Chronometer
- ImageButton
- ImageView
- ProgressBar
- TextView
- ViewFlipper
- ListView
- GridView
- StackView
- AdapterViewFlipper

>Descendants of these classes are not supported.不支持这些类的后代

也就是说，只有这些控件能在布局中使用，其他的控件无法再布局中使用，包括这些类的子类，也就是说Widget中的布局是有局限的，不能像再Activity那么为所欲为0.0 而且由于widget运行在桌面进程中，而不是程序主进程，所以对UI的一切操作需要用到RemoteView类来进行操作。

```java
RemoteViews view = new RemoteViews(context.getPackageName(), R.layout.widget);
```

首先拿到一个view对象，然后通过view的一系列方法进行操作

####RemoteViews

Remotes类提供了对layout的更新和控制，我们可以通过view的一些列方法进行有限的对layout的一些操控，例如设置id为text的TextView的文本：

```java
view.setTextViewText(R.id.text, "Hello World");
```

例如设置这个TextView的点击事件监听

```java
//设置打开系统时钟
view.setOnClickPendingIntent(R.id.clock_layout, PendingIntent.getActivity(context, 2, new Intent().setComponent(new ComponentName("com.android.deskclock", "com.android.deskclock.DeskClock")), PendingIntent.FLAG_UPDATE_CURRENT));
```

第一个参数是需要设置的控件ID，第二个参数是一个PendingIntent，在Widget中设置监听无法直接用Intent传递，需要用PendingIntent来进行传递。

其格式大概为 set + (控件名) + 控件属性
格式不绝对，仅供参考，所以view中没有提供的方法，就无法进行修改。

####PendingIntent

使用这样一个方法来获取我们需要的一个基本的PendingIntent（这个方法我自己实现的，非官方）

```java
    //给按钮返回PendingIntent
    private PendingIntent getClickIntent(Context context, int widgetId, int viewId, int requestCode, String action) {
        //拿到管理对象
        AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
        //pendingintent中需要的intent，绑定这个类和当前context
        Intent i = new Intent(context, MyWidget.class);
        //设置action
        i.setAction(action); //设置更新动作
        //设置bundle
        Bundle bundle = new Bundle();
        //将widgetId放进bundle
        bundle.putInt(AppWidgetManager.EXTRA_APPWIDGET_ID, widgetId);
        //放进需要设置的viewId
        bundle.putInt("Button", viewId);
		//由于AppwidgetProvider是BroadCastReceiver的子类，所以这里的pendingintent来自于getBroadcast
        i.putExtras(bundle);
        PendingIntent pendingIntent = PendingIntent.getBroadcast(context, requestCode, i, PendingIntent.FLAG_UPDATE_CURRENT);
        return pendingIntent;
    }
```

五个参数分别是当前的context，widget的id号码，需要设置的view的id号码，返回码，action，其余代码详情见注释

####RemoteViewsService

这个是Service的子类，多用于对“GridView，ListView”等控件进行管理。

这么进行设置

- 设置一个pendingintent模板

```java
views.setPendingIntentTemplate(R.id.listView, getClickIntent(context, appWidgetIds[0], R.id.listView, 1, "com.longlong.myblogwidget.COLLECTION_VIEW"));
```

根据后面这个action来在onReceiver对其事件做出响应

- 设置Adapter

```java
views.setRemoteAdapter(R.id.listView, new Intent(context, MyWidgetListViewService.class));
```

在RemoteViewsService中可以实现控制

```java
public class MyWidgetListViewService extends RemoteViewsService {
    @Override
    public RemoteViewsFactory onGetViewFactory(Intent intent) {
        return new ViewFactory();
    }

    private class ViewFactory implements RemoteViewsFactory {

        private ArrayList<String> mDataList = new ArrayList<String>();

        //onCreate方法在类创建的时候调用
        @Override
        public void onCreate() {
            for (int i = 0; i < 5; i++) {
                mDataList.add("Item " + String.valueOf(i));
            }
        }

        //此方法在类
        @Override
        public void onDataSetChanged() {

        }

        @Override
        public void onDestroy() {

        }

        @Override
        public int getCount() {
            return 0;
        }

        @Override
        public RemoteViews getViewAt(int position) {
            return null;
        }

        @Override
        public RemoteViews getLoadingView() {
            return null;
        }

        @Override
        public int getViewTypeCount() {
            return 0;
        }

        @Override
        public long getItemId(int position) {
            return 0;
        }

        @Override
        public boolean hasStableIds() {
            return false;
        }
    }
}
```

该类返回一个对象，这个对象中的实现用来控制listView（此类类似BaseAdapter）

```java
    @Override
    public RemoteViews getViewAt(int position) {
	//这里就类似getView函数了,但是实现点击同样是需要remoteview的
        RemoteViews views = new RemoteViews(mContext.getPackageName(), R.layout.item_layout);
        views.setTextViewText(R.id.text, data.get(position));
	    //设置点击监听,反应事件在Widget中的onReceived中实现
        views.setOnClickFillInIntent(R.id.item_layout, new Intent().putExtra("POSITION", position));
        return views;
    }
```