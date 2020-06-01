---
title: Android手机应用开发（五） | AppWidget 使用
urlname: android5
tags:
  - Android
  - AppWidget
copyright: true
category: Android
abbrlink: 43131
date: 2018-10-25 20:29:08
---

> 实验目的
>
> 1. 复习 Broadcast 编程基础。
> 2. 复习动态注册 Broadcast 和静态注册 Broadcast 。
> 3. 掌握 AppWidget 编程基础。



**先上效果图**





![GIF](https://blog.janking.cn/post/android5/GIF-1540489328401.gif)

<!-- more --> 

## 创建一个AppWidget

`File` -> `New` -> `Widget` -> `AppWidget`

自定义命名，我命名为`MyWidget`

![1540470758342](https://blog.janking.cn/post/android5/1540470758342.png)

就会多出三个文件

![1540470892309](https://blog.janking.cn/post/android5/1540470892309.png)

`my_widget_info.xml`这是小部件的一些属性设置

```xml
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:initialKeyguardLayout="@layout/my_widget"
    android:initialLayout="@layout/my_widget"
    android:minWidth="300dp"
    android:minHeight="50dp"
    android:previewImage="@mipmap/full_star"
    android:resizeMode="horizontal|vertical"
    android:updatePeriodMillis="86400000"
    android:widgetCategory="home_screen|keyguard"></appwidget-provider>
```

`my_widget.xml`这是小部件的外观

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="300dp"
    android:layout_height="80dp"
    android:background="#0000">

    <ImageView
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:layout_centerVertical="true"
        android:id="@+id/widget_image"
        android:src="@mipmap/full_star"/>
    <TextView
        android:id="@+id/appwidget_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true"
        android:contentDescription="@string/appwidget_text"
        android:text="@string/appwidget_text"
        android:textColor="#ffffff"
        android:textSize="15sp"
        android:textStyle="bold" />

</RelativeLayout>
```

下面就主要处理好`AppWidgetProvider`中的这些方法了

![1540471542308](https://blog.janking.cn/post/android5/1540471542308.png)

## 通过静态广播更新

静态比较好理解，就是直接把管理`Widget`的`java`类放进`Android`的`manifest`配置文件中，以后有什么需要更新的就去找这个类

打开`AndroidManifest.xml`，添加

```
<receiver android:name=".MyWidget">
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_STARTUP"/>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
    </intent-filter>
    <meta-data
        android:name="android.appwidget.provider"
        android:resource="@xml/my_widget_info" />
</receiver>
```

为了让小部件显示“`当前没有任何信息`”，修改`MyWidget.java`，添加

```java
static void updateAppWidget(Context context, AppWidgetManager appWidgetManager,
                            int appWidgetId) {

    CharSequence widgetText = context.getString(R.string.appwidget_text);
    // Construct the RemoteViews object
    RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.my_widget);
    views.setTextViewText(R.id.appwidget_text, widgetText);
    // Instruct the widget manager to update the widget
    appWidgetManager.updateAppWidget(appWidgetId, views);
}

@Override
public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
    // There may be multiple widgets active, so update all of them
    for (int appWidgetId : appWidgetIds) {
        updateAppWidget(context, appWidgetManager, appWidgetId);
        RemoteViews updateView = new RemoteViews(context.getPackageName(), R.layout.my_widget);//实例化RemoteView,其对应相应的Widget布局
        Intent i = new Intent(context, SecondActivity.class);
        PendingIntent pi = PendingIntent.getActivity(context, 0, i, PendingIntent.FLAG_UPDATE_CURRENT);
        updateView.setOnClickPendingIntent(R.id.widget_image, pi); //设置点击事件
        ComponentName me = new ComponentName(context, MyWidget.class);
        appWidgetManager.updateAppWidget(me, updateView);

    }
}
```

为了让`SecondActivity`一打开就会在更新小部件，显示“`今日推荐 xxx`”，打开`SecondActivity.java`，添加

```java
@Override
protected void onStart(){
    super.onStart();
    //broadcast
    Bundle bundle = new Bundle();
    bundle.putSerializable("widget_startup", mAdapter.getItem(new Random().nextInt(mAdapter.getItemCount())));//返回一个0到n-1的整数
    Intent widgetIntentBroadcast = new Intent();
    widgetIntentBroadcast.setAction("android.appwidget.action.APPWIDGET_STARTUP");
    widgetIntentBroadcast.setComponent(new ComponentName("com.janking.sysuhealth", "com.janking.sysuhealth.MyWidget"));
    widgetIntentBroadcast.putExtras(bundle);
    sendBroadcast(widgetIntentBroadcast);

    //很关键，因为不知道为什么有时候会调用onCreate函数,然后一切都变了
    Food f = (Food)getIntent().getSerializableExtra("widget_favorite");
    if(f != null )
        EventBus.getDefault().post(f);
}
```

> 因为`SecondActivity`设置了单例模式，所以在`onCreate`发送广播不一定成功，而每次唤醒这个`Activity`时`onStart()`总是被调用



然后处理接收广播的事件，打开`MyWidget.java`，添加

```java
@Override
public void onReceive(Context context, Intent intent) {
    super.onReceive(context, intent);
    AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
    if(intent.getAction().equals("android.appwidget.action.APPWIDGET_STARTUP")){//初始化
        Bundle bundle = intent.getExtras();
        Food food = (Food)bundle.getSerializable("widget_startup");
        //获取pendingIntent
        //这里继续使用Click_Food关键词，就不用再Detail里加一个判断语句了
        bundle.putSerializable("Click_Food", food);
        Intent mainIntent = new Intent(context, Detail.class);
        mainIntent.putExtras(bundle);
        PendingIntent myPendingIntent = PendingIntent.getActivity(context, 0, mainIntent, PendingIntent.FLAG_UPDATE_CURRENT);
        RemoteViews updateView = new RemoteViews(context.getPackageName(), R.layout.my_widget);//实例化RemoteView,其对应相应的Widget布局
        updateView.setOnClickPendingIntent(R.id.widget_image, myPendingIntent); //设置点击事件
        updateView.setTextViewText(R.id.appwidget_text, "今日推荐 " + food.getName());
        ComponentName cn = new ComponentName(context, MyWidget.class);
        appWidgetManager.updateAppWidget(cn, updateView);
    }
}
```

就可以实现点击小部件跳转到`Detail`详情页面了，其中的数据传送跟之前的差不多，都是通过`Intent`中的`Bundle`

## 通过动态广播更新

本来我是打算学动态广播的办法，新创建一个java类在代码中动态注册，确实那么做了，但是一直没有效果，想了想，如果新建一个继承`AppWidgetProvider`的话，又要写`onUpdate`,`onReceive`等方法，如果在多写几个这样的类，岂不是乱套了，通知该由谁接收？

仔细一想之后，还是放在之前的`DynamicReceiver`里面吧，毕竟只要处理一下显示和`pendingIntent`就好了



所以，打开`DynamicReceiver.java`文件，在`if`语句后面添加

```java
//if(....)
    //......
else if(intent.getAction().equals("android.appwidget.action.APPWIDGET_UPDATE")){//更新
    AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
    Bundle bundle = intent.getExtras();
    Food food = (Food)bundle.getSerializable("widget_favorite");
    //获取pendingIntent
    //这里继续使用Click_Food关键词，就不用再Detail里加一个判断语句了
    Intent mainIntent = new Intent(context, SecondActivity.class);
    mainIntent.putExtras(bundle);
    PendingIntent myPendingIntent = PendingIntent.getActivity(context, 0, mainIntent, PendingIntent.FLAG_UPDATE_CURRENT);
    RemoteViews updateView = new RemoteViews(context.getPackageName(), R.layout.my_widget);//实例化RemoteView,其对应相应的Widget布局
    updateView.setOnClickPendingIntent(R.id.widget_image, myPendingIntent); //设置点击事件
    updateView.setTextViewText(R.id.appwidget_text, "已收藏 " + food.getName());
    ComponentName cn = new ComponentName(context, MyWidget.class);
    appWidgetManager.updateAppWidget(cn, updateView);
    //EventBus.getDefault().post(food);//这里不用EventBus了，因为重新打开SecondActivity后会调用OnCreate函数
}
```

上面是处理了点击收藏按钮之后Widget的更新

现在在`Detail.java`里面添加注册动态广播

```java
//注册广播
//IntentFilter dynamic_filter = new IntentFilter();
//dynamic_filter.addAction("com.janking.sysuhealth.myapplication2.MyDynamicFilter");    //添加动态广播的Action
dynamic_filter.addAction("android.appwidget.action.APPWIDGET_UPDATE");
//dynamicReceiver = new DynamicReceiver();
//registerReceiver(dynamicReceiver, dynamic_filter);    //注册自定义动态广播消息
```

并在收藏按钮的点击事件中添加

```java
collect.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
		//省略其它
        Bundle bundleWidget = new Bundle();
        bundleWidget.putSerializable("widget_favorite", display_food);//返回一个0到n-1的整数
        Intent intentBroadcastWidget = new Intent("android.appwidget.action.APPWIDGET_UPDATE"); //定义Intent
        intentBroadcastWidget.putExtras(bundleWidget);
        sendBroadcast(intentBroadcastWidget);

    }
```

## 笔记

上次把动态广播取消注册放在了`OnPause`里面，然后运行的时候抛出了异常

```java
java.lang.RuntimeException: Unable to pause activity {com.janking.sysuhealth.DetailActivity}: java.lang.IllegalArgumentException: Receiver not registered
```

[参考了这篇博客](http://www.cnblogs.com/fengzhblog/archive/2013/07/16/3193067.html)（但其实情况不一样）

发现应该在`onDestory`里面取消注册，把`onPause`删了，然后像这样

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    unregisterReceiver(dynamicReceiver);
}
```



完美！

好吧，其实不完美，Widget好多跳转来跳转去的复杂东西没有考虑好，就这样吧，核心功能实现了就好！

<!-- more --> 