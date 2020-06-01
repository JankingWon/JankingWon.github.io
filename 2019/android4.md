---
title: Android手机应用开发（四） | Broadcast使用和Notification、EventBus编程基础
urlname: android4
tags:
  - Android
  - Broadcast
  - Notification
  - EventBus
copyright: true
category: Android
abbrlink: 44330
date: 2018-10-17 20:20:00
---

> 实验目的
>
> 1. 掌握 Broadcast 编程基础。
> 2. 掌握动态注册 Broadcast 和静态注册 Broadcast。
> 3. 掌握Notification 编程基础。
> 4. 掌握 EventBus 编程基础。

<!-- more --> 

## 设置Activity为单例模式

`manifests`中添加

`android:launchMode="singleInstance"`



> 还有这几种情况：
>
> (1)`standard`：每次激活`Activity`时(`startActivity`)，都创建`Activity`实例，并放入任务栈；
>
> (2)`singleTop`：如果某个`Activity`自己激活自己，即任务栈栈顶就是该`Activity`，则不需要创建，其余情况都要创建`Activity`实例；
>
> (3)`singleTask`：如果要激活的那个`Activity`在任务栈中存在该实例，则不需要创建，只需要把此`Activity`放入栈顶，并把该`Activity`以上的`Activity`实例都`pop`；
>
> (4)`singleInstance`：如果应用1的任务栈中创建了`MainActivity`实例，如果应用2也要激活`MainActivity`，则不需要创建，两应用共享该`Activity`实例；

## Broadcast使用

`BroadcastReceiver`（广播接收器），属于 `Android` 四大组件之一

注册的方式分为两种：**静态注册**、**动态注册**

### 静态注册

#### 1.注册广播

创建一个`java`类`StaticReceiver`

```java
public class StaticReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction().equals("com.janking.sysuhealth.myapplication2.MyStaticFilter")){
            //代表捕获到需要的广播，内容稍后填充
        }
    }

}
```

然后在配置文件`App->manifests->AndroidManifest.xml`中加上

```xml
<receiver android:name=".StaticReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="com.janking.sysuhealth.myapplication2.MyStaticFilter" />
    </intent-filter>
</receiver>
```

> 这里的`com.janking.sysuhealth.myapplication2.MyStaticFilter`理论上可以自定义，只要保持一致就可以（后面还会出现）

#### 2.发送广播

这里要实现打开页面就会弹出通知并随机出现一个`Food`的名字，点击跳到其详情页面

所以在`Food`列表的`onCreate`函数(即`SecondActivity`)中加上

```java
//broadcast
Bundle bundle = new Bundle();
//把整个Food类变量放进bundle
bundle.putSerializable("broadcast_startup", mAdapter.getItem(new Random().nextInt(mAdapter.getItemCount())));//返回一个0到n-1的整数
Intent intentBroadcast = new Intent("com.janking.sysuhealth.myapplication2.MyStaticFilter"); //定义Intent
//下面这一句相当关键，这是Android8.0之后之后的写法
intentBroadcast.setComponent(new ComponentName("com.janking.sysuhealth", "com.janking.sysuhealth.StaticReceiver"));
intentBroadcast.putExtras(bundle);
sendBroadcast(intentBroadcast);
```

这里我用的API是28，即`compileSdkVersion 28`，跟之前的版本可能会**不一样**！

[关于Android SDK里的compileSdk、minSdk、targetSdk](https://blog.csdn.net/DovSnier/article/details/79870590)

#### 3.接收广播

要认识到，广播其实也是以`Intent`的方式传过来的

所以还是这样解析传过来的`Food`

```java
Bundle bundle = intent.getExtras();
Food food = (Food)bundle.getSerializable("broadcast_startup");
```

#### 4.发送通知

不管什么时候，看到`发送`、`接收`、`跳转`这些字眼就只要有**数据的传送**，一般用的是`Intent`，这里要用`PendingIntent`

> 根据字面意思就知道是延迟的`intent`，主要用来在某个事件完成后执行特定的`Action`。`PendingIntent`包含了`Intent`及`Context`，所以就算`Intent`所属程序结束，`PendingIntent`依然有效，可以在其他程序中使用。
> 常用在通知栏及短信发送系统中。



创建`PendingIntent`

```java
//获取pendingIntent
//这里继续使用Click_Food关键词，就不用再Detail里加一个判断语句了
bundle.putSerializable("Click_Food", food);
Intent mainIntent = new Intent(context, Detail.class);
mainIntent.putExtras(bundle);
PendingIntent mainPendingIntent = PendingIntent.getActivity(context, 0, mainIntent, PendingIntent.FLAG_UPDATE_CURRENT);
```

又提到`Android SDK`版本的问题了，这里是`8.0(API26)`以上的使用方法，理论上低版本的方法更容易找到，但是我们目光还是要向前看（/滑稽）

创建通知`Notification`

```java
//获取状态通知栏管理
NotificationManager manager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
//channel
String CHANNEL_ID = "channel_01";
NotificationChannel mChannel = null;
if (mChannel == null) {
    String name = "my_channel_01";//渠道名字
    String description = "my_package_first_channel"; // 渠道解释说明
    //HIGH或者MAX才能弹出通知到屏幕上
    int importance = NotificationManager.IMPORTANCE_HIGH;
    mChannel = new NotificationChannel(CHANNEL_ID, name, importance);
    mChannel.setDescription(description);
    mChannel.enableLights(true); //是否在桌面icon右上角展示小红点
    manager.createNotificationChannel(mChannel);
}
//notification
NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(context, CHANNEL_ID)
        .setSmallIcon(R.mipmap.empty_star)
        .setContentTitle("今日推荐")
        .setTicker("您有一条新消息")   //通知首次出现在通知栏，带上升动画效果的
        .setContentText(food.getName())
        .setDefaults(Notification.DEFAULT_ALL)
        .setAutoCancel(true)   //设置这个标志当用户单击面板就可以让通知将自动取消
        .setContentIntent(mainPendingIntent);
//display
Notification notification = mBuilder.build();
manager.notify(0,notification);
```

> (以上三段代码都是放入`StaticReceiver`中的`if`语句中)

`mChannel = new NotificationChannel(CHANNEL_ID, name, importance);`

和

`NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(context, CHANNEL_ID)`

一定要用**同一个**`CHANNEL_ID`,不然就会出现下列错误

![1539870233846](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android4/1539870233846.png)

------

那么现在，如果点击通知跳转到食品详情页面，`Detail.java`需要更改什么吗?

**不需要**，这就像从食品列表跳转到详情一样，解析`Food`就行了，而且我很偷懒地把从`Notification`中发出的`PendingIntent`中的`Food`关键词也设置成`Click_Food`，对`Detail`来讲，它就像从列表跳过来一样！

### 动态注册

> 其实在Android 8.0之后是不建议用静态注册的，毕竟不灵活，所以还是关注下动态注册吧

#### 1.注册广播

创建类`DynamicReceiver`

```java
public class DynamicReceiver extends BroadcastReceiver {
    private static final String DYNAMICACTION = "com.janking.sysuhealth.myapplication2.MyDynamicFilter";
    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction().equals(DYNAMICACTION)) {    //动作检测
        //稍后填充
        }
    }
}
```

在需要发送广播的类中（这里是`Detail`）添加

```java
//注册广播
IntentFilter dynamic_filter = new IntentFilter();
dynamic_filter.addAction("com.janking.sysuhealth.myapplication2.MyDynamicFilter");    //添加动态广播的Action
dynamicReceiver = new DynamicReceiver();
registerReceiver(dynamicReceiver, dynamic_filter);    //注册自定义动态广播消息
```

> 注意两个地方的`Action`要一致（下面还会出现）

#### 2.发送广播

这里发送广播的激发事件是点击了Detail中的收藏图标

![1539869840890](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android4/1539869840890.png)

所以修改`collect.setOnClickListener`为

```java
collect.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        if(!display_food.getFavorite()){
            change++;
            Toast.makeText(Detail.this, "已收藏", Toast.LENGTH_SHORT).show();
        }
        else
            Toast.makeText(Detail.this, "重复收藏", Toast.LENGTH_SHORT).show();
        display_food.setFavorite(true);
        //发送广播
        Bundle bundle = new Bundle();
        bundle.putSerializable("broadcast_favorite", display_food);//返回一个0到n-1的整数
        Intent intentBroadcast = new Intent("com.janking.sysuhealth.myapplication2.MyDynamicFilter"); //定义Intent
        //下面一句用来发送静态广播，这里不需要
        //intentBroadcast.setComponent(new ComponentName("com.janking.sysuhealth", "com.janking.sysuhealth.DynamicReceiver"));
        intentBroadcast.putExtras(bundle);
        sendBroadcast(intentBroadcast);

    }
});
```

#### 3.接收广播

在`DynamicReceiver`中的`if`语句中添加

```java
Bundle bundle = intent.getExtras();
Food food = (Food)bundle.getSerializable("broadcast_favorite");
```

#### 4.发送通知

再添加

```java
//获取pendingIntent
Intent mainIntent = new Intent(context, SecondActivity.class);
mainIntent.putExtras(bundle);
PendingIntent mainPendingIntent = PendingIntent.getActivity(context, 0, mainIntent, PendingIntent.FLAG_UPDATE_CURRENT);

//获取状态通知栏管理
NotificationManager manager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
//channel
String CHANNEL_ID = "channel_02";
NotificationChannel mChannel = null;
//不用重复赋值
if (mChannel == null) {
    String name = "my_channel_02";//渠道名字
    String description = "my_package_second_channel"; // 渠道解释说明
    //HIGH或者MAX才能弹出通知到屏幕上
    int importance = NotificationManager.IMPORTANCE_HIGH;
    mChannel = new NotificationChannel(CHANNEL_ID, name, importance);
    mChannel.setDescription(description);
    mChannel.enableLights(true); //是否在桌面icon右上角展示小红点
    manager.createNotificationChannel(mChannel);
}
//notification
NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(context, CHANNEL_ID)
        .setSmallIcon(R.mipmap.full_star)
        .setContentTitle("已收藏")
        .setTicker("您有一条新消息")   //通知首次出现在通知栏，带上升动画效果的
        .setContentText(food.getName())
        .setDefaults(Notification.DEFAULT_ALL)
        .setAutoCancel(true)   //设置这个标志当用户单击面板就可以让通知将自动取消
        .setContentIntent(mainPendingIntent);
//display
Notification notification = mBuilder.build();
manager.notify(0,notification);
```

> 其实本质上这两个广播都是一样的，只是注册方式不一样，处理方式都是差不多的

#### 5. 取消注册

这个是一定要做的，不然会出现**内存泄漏**的错误

```
E/ActivityThread: Activity com.janking.sysuhealth.Detail has leaked IntentReceiver com.janking.sysuhealth.DynamicReceiver@c408803 that was originally registered here. Are you missing a call to unregisterReceiver()?
```

![1539871075001](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android4/1539871075001.png)

> **注意：**动态广播最好在`Activity` 的 `onResume()`注册、`onPause()`注销，当然我这里是在`Oncreate()`中注册的也行

所以在`Detail`中添加

```java
@Override
protected void onPause() {
    super.onPause();
    unregisterReceiver(dynamicReceiver);
}
```

> 重复注册、重复注销也不允许

------



## EventBus的简单使用

其实到这里，数据同步的问题还没有解决

当点击收藏弹出消息时，点击消息只能进去`SecondActivity`（即**食品**列表）界面，现在需要它显示为**收藏食品**列表界面，并且能够更新某些食品

当然直接用广播传来的数据是可以的，不过还是要学会使用多种方法（因为其实用广播处理的话是有点难…….）

### 1.添加依赖

在`Gradle Scripts ->build.gradle(Module: app)`中的`dependencies`添加

`implementation 'org.greenrobot:eventbus:3.0.0'`

### 2.声明订阅方法

在需要**接收**数据的`Activity`（这里是`SecondActivity`）中添加

```java
//EventBus
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessageEvent(Food f) {
    //change RecyclerView
    mAdapter.updateData(f.getName(), f.getFavorite());
    //change listview
    myListViewAdapter.addNewItem(f);
    //change floatbutton
    isHome = false;
    mRecyclerView.setVisibility(View.INVISIBLE);
    mListView.setVisibility(View.VISIBLE);
    mButton.setImageResource(R.mipmap.mainpage);
};
```

> 不要添加到`onCreate()`方法或者其它方法中，要直接加到`Activity`类中，当做一个成员。
>
> 不然会出现错误`@Override “Annotations are not allowed here”`

这里的处理大概就是更新食品总列表中的数据（不可见）以及收藏食品列表中的元素（课件），而且改变下面的`FloatingButton`，并且使`RecyclerView`不可见，`ListView`可见

### 3.订阅

在`SecondActivity`中添加（比如在`onCreate()`)

```java
EventBus.getDefault().register(this);
```

### 4.发送数据

这里就是在点击Detail中的通知后返回到SecondActivity页面，想来想去，也只有一个Food类型的数据要传过来，那就在之前的`DynamicReceiver`中的`if`语句中添加一句

```java
EventBus.getDefault().post(food);
```

就能传送数据了，真的是超级简单啊！！！

### 5.解决列表重复的问题

在`Detail`页面中多次点击收藏图标的话，虽然我之前做了处理，弹出的`Toast`会显示“**重复收藏**”，但是每次点击都会发送广播，每次发送广播都会有一个post(food)的动作，即使不跳过去看，但是其实数据已经右`EventBus`传到了``SecondActivity`了，并且也已经处理了。

所以我想了个办法，对这一句`myListViewAdapter.addNewItem(f);`做做手脚，找到`MyListViewAdapter`中的`MyListViewAdapter`，改为

```java
public void addNewItem(Food f) {
    if(list == null) {
        list = new ArrayList<>();
    }
    for(Food i : list){
        if(i.getName().equals(f.getName())){
            notifyDataSetChanged();
            return;
        }
    }
    list.add(f);
    notifyDataSetChanged();
}
```

就**不会重复**啦！



## 效果：（AVD实在有点卡）

![broadcast](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android4/broadcast.gif)



------

## 参考资料

[AVD,Guest isn't online after 7 seconds 问题](https://blog.csdn.net/DovSnier/article/details/79870590)

[安卓8.0系统notification适配Failed to post notification on](https://www.lanpan.cn/article-666-1.html)

[android show notification with a popup on top of any application](ttps://stackoverflow.com/questions/29949501/android-show-notification-with-a-popup-on-top-of-any-application)

<!-- more --> 