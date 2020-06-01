---
title: Android手机应用开发（八） | 制作简单音乐播放器
urlname: android8
tags:
  - Android
  - MediaPlayer
  - Handler
  - Service
  - 多线程
copyright: true
category: Android
abbrlink: 25302
date: 2018-11-27 12:35:19
---

> 实验目的
>
> 1. 学会使用MediaPlayer
> 2. 学会简单的多线程编程，使用Handler更新UI
> 3. 学会使用Service进行后台工作
> 4. 学会使用Service与Activity进行通信

<!-- more --> 

## 效果预览

![gif5新文件](https://blog.janking.cn/post/android8/gif5新文件.gif)

## 布局

![1543298569700](https://blog.janking.cn/post/android8/1543298569700.png)

### 进度条的布局

如何实现让进度条占满`当前时间`和`全部时间`中间的部分呢？

- 如果使用`match_parent`，右边的`全部时间`又显示不了

- 如果使用`wrap_content`，又不能填充满

- 如果自定义`dp`值，不同尺寸显示又会不一样



这就利用了`LinearLayout`的特点，只需设置中间进度条的`layout_weight = 1`就好了，它就会自动延伸到右边最远处（不占据别的控件）

![1543298911582](https://blog.janking.cn/post/android8/1543298911582.png)

### 圆形ImageView

这里使用了`github`上的开源控件：[链接](https://github.com/hdodenhof/CircleImageView)

1. 添加依赖`implementation 'de.hdodenhof:circleimageview:2.2.0'`

2. 在`xml`中使用

   ```xml
   <de.hdodenhof.circleimageview.CircleImageView
       xmlns:app="http://schemas.android.com/apk/res-auto"
       android:id="@+id/profile_image"
       android:layout_width="match_parent"
       android:layout_height="290dp"
       android:layout_marginTop="30dp"
       android:src="@drawable/img"
       app:layout_constraintTop_toTopOf="parent" />
   ```

## Service的使用

`Service`(服务)是一种可以在后台执行长时间运行操作而没有用户界面的应用组件。

服务可由其他应用组件启动（如`Activity`），服务一旦被启动将在后台一直运行，即使启动服务的组件（`Activity`）
已销毁也不受影响。

### 如何启动`Service`
1. 通过`startService`启动

   `startService()`启动和`stopService()`关闭服务，`Service`与访问者之间基本不存在太多关联，因此`Service`和访问者之间无法通讯和数据交换。

2. 通过`bindService`启动

   用于`Service`和访问者之间需要进行方法调用或数据交换的情况

### 注册Service

在`manifests`里面的`application`里，添加

```xml
<service android:name=".MusicService" android:exported="true"/>
```

### 创建Service

右键 -> `New` -> `Service` -> `Service`，取名为`MusicService`(跟注册时一致)

在`Service`里添加成员

```java
//用来跟Activity进行绑定
public final IBinder binder = new MyBinder();
//媒体播放类
public MediaPlayer mp =  new MediaPlayer();
//对Service控制的不同数字码
private final int PLAY_CODE = 1, STOP_CODE = 2, SEEK_CODE = 3, NEWMUSIC_CODE = 4, CURRENTDURATION_CODE = 5, TOTALDURATION = 6;
//重写onTransact方法，对不同的CODE做出不同的反应
public class MyBinder extends Binder {
        @Override
        protected boolean onTransact(int code, @NonNull Parcel data, @Nullable Parcel reply, int flags) throws RemoteException {
            switch (code) {
                //service solve
                case PLAY_CODE:
                    play_pause();
                    break;
                case STOP_CODE:
                    stop();
                    break;
                case SEEK_CODE:
                    mp.seekTo(data.readInt());
                    break;
                case NEWMUSIC_CODE:
                    newMusic(Uri.parse(data.readString()));
                    reply.writeInt(mp.getDuration());
                case TOTALDURATION:
                    reply.writeInt(mp.getDuration());
                    break;
                case CURRENTDURATION_CODE:
                    reply.writeInt(mp.getCurrentPosition());
                    break;
            }
            return super.onTransact(code, data, reply, flags);
        }
    }
```

> 这里的`Environment.getExternalStorageDirectory()`指的外部存储不是扩展卡中的存储，而是相对程序来讲，不是程序的`InternalStorage`，而是本机的通用存储

重写`onBind`方法

```java
@Override
public IBinder onBind(Intent intent) {
    try {
            mp.setDataSource(Environment.getExternalStorageDirectory() + "/data/山高水长.mp3");
            mp.prepare();
        } catch (IOException e) {
            Log.e("prepare error", "getService: " + e.toString());
        }
        return binder;
}
```

> 这里返回的`binder`就相当于一个`Service`组件所返回的代理对象，`Service`允
> 许客户端通过该`IBinder`对象来访问`Service`内部的数据，实现客户端与`Service`之间的通信

编写一些简单的控制方法

```java
public void play_pause() {
        if (mp.isPlaying()) {
            mp.pause();
        } else {
            mp.start();
        }
    }
    public void stop() {
        if (mp != null) {
            mp.stop();
            try {
                mp.prepare();
                mp.seekTo(0);
            } catch (Exception e) {
                Log.d("stop", "stop: " + e.toString());
            }
        }
    }
    public void newMusic(Uri uri){
            try{
                mp.reset();
                mp.setDataSource(this, uri);
                mp.prepare();
            }
       catch (Exception e){
            Log.d("New Music", "new music: " + e.toString());
        }
    }

```

重写`onDestory`方法

```java
@Override
public void onDestroy() {
    super.onDestroy();
    if(mp!= null){
        //release之前一定要reset，不然会报下面的错
        //W/MediaPlayer(7564): mediaplayer went away with unhandled events
        mp.reset();
        mp.release();
    }
}
```

### 绑定Service

在`MainActivity`中添加成员变量

```java
private IBinder mBinder;
private final int PLAY_CODE = 1, STOP_CODE = 2, SEEK_CODE = 3, NEWMUSIC_CODE = 4, CURRENTDURATION_CODE = 5, TOTALDURATION = 6;
private int total_duration = 0;
```

添加绑定成功的操作

```java
private ServiceConnection sc = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        mBinder = service;
        //绑定成功时进行的操作
        //还有一些没有写出来
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
        ms = null;
        //结束绑定时进行的操作
    }
};
```

使用下面代码进行绑定

```java
Intent intent = new Intent(this, MusicService.class);
bindService(intent, sc, BIND_AUTO_CREATE);
```

> 这就像用`Intent`开启一个`Activity`一样，只不过这里是开启`Service`
>
> 但是具体的操作还是在`onServiceConnected`中完成

## MediaPlayer的使用

绑定好了之后就可以通过`ms`访问`mp`了

但是现在直接调用`ms.play()`会报错

因为没有权限去访问预先设置的音频文件

### 添加文件访问权限

在`Android6.0`之前，我们只需要在`AndroidManifest.xml`文件中直接添加权限即可

但是在`Android6.0`之后，我们只在`AndroidManifest.xml`文件中配置是不够的，还需要在`Java`代码中进行动态获取权限。

![](https://blog.janking.cn/post/android8/clip_image001-1543301104114.png)

#### 静态添加

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

#### 动态添加

```java
/读写权限
private static String[] PERMISSIONS_STORAGE = {
        Manifest.permission.READ_EXTERNAL_STORAGE,
        Manifest.permission.WRITE_EXTERNAL_STORAGE};
//请求状态码
private static int REQUEST_PERMISSION_CODE = 1;
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		//获取权限
        ActivityCompat.requestPermissions(this, PERMISSIONS_STORAGE, REQUEST_PERMISSION_CODE);
		......
}
//成功获取权限的回调方法，在这里
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == REQUEST_PERMISSION_CODE) {
            //其实应该放在这里开始绑定，因为只有获取权限成功后绑定才能正确让音乐播放
            Intent intent = new Intent(this, MusicService.class);
            bindService(intent, sc, BIND_AUTO_CREATE);
        }
}
```

### onServiceConnected的补充

绑定成功后，需要更新进度条的长度，音乐总时间和当前进度（初始化为0）

```java
//记得在onCreate外面添加类成员变量
private final int PLAY_CODE = 1, STOP_CODE = 2, SEEK_CODE = 3, NEWMUSIC_CODE = 4, CURRENTDURATION_CODE = 5, TOTALDURATION = 6;
private int total_duration = 0;

@Override
public void onServiceConnected(ComponentName name, IBinder service) {
    mBinder = service;
    //与服务通信
    Parcel data = Parcel.obtain();
    Parcel reply = Parcel.obtain();
    try {
        mBinder.transact(TOTALDURATION, data, reply, 0);
    }catch (Exception e){
        Log.d("SERVICE CONNECTION", "onServiceConnected: " + e.toString());
    }
    total_duration = reply.readInt();
    seekbar.setProgress(0);
    seekbar.setMax(total_duration);
    //两种方法实现毫秒转时间
    //total_time.setText(time.format(new Date(ms.mp.getDuration())));
    total_time.setText(DateFormat.format(time_format, total_duration));
    current_time.setText(time.format(new Date(0)));
}
```

#### 毫秒格式化为时间显示

`ms.mp.getDuration()`返回的是一个`long`型的歌曲毫秒数，需要格式化显示为时间的形式，如：12:12

- 方法一：

  ```java
  private SimpleDateFormat time = new SimpleDateFormat("mm:ss");
  total_time.setText(time.format(new Date(ms.mp.getDuration())));
  ```

- 方法二：

  ```java
  private String time_format = "mm:ss";
  total_time.setText(DateFormat.format(time_format, (long)ms.mp.getDuration()));
  ```

### 播放/暂停按钮

```java
play_pause.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        //与服务通信
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        try{
            mBinder.transact(PLAY_CODE, data, reply, 0);
        }catch (RemoteException e){
            Log.e("STOP:", "onClick: " + e.toString() );
        }
        if(isPlay){
            isPlay = false;
            play_pause.setImageResource(R.mipmap.play);
        }
        else {
            isPlay = true;
            isStop = false;
            play_pause.setImageResource(R.mipmap.pause);
            //开始监听
            myThread.run();
        }
    }
});
```

### 停止按钮

```java
stop.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
       isPlay = false;
       isStop = true;
       play_pause.setImageResource(R.mipmap.play);
        seekbar.setProgress(0);
        current_time.setText(time.format(0));
        imageView.setRotation(0);
        //与服务通信
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        try{
            mBinder.transact(STOP_CODE, data, reply, 0);
        }catch (RemoteException e){
            Log.e("STOP:", "onClick: " + e.toString() );
        }
    }
});
```

### 图片的旋转

有很多方法，这里用一种比较简单的设置角度的方法

```java
imageView.setPivotX(imageView.getWidth()/2);
imageView.setPivotY(imageView.getHeight()/2);//支点在图片中心
//表示顺时针旋转90度
imageView.setRotation(90);
```

### 拖动条事件

```java
seekbar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
    @Override
    public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
        //判断是否来自用户，因为后面还要设置播放的时候进度条跟着变化，如果那样的变化也调用这个方法的话进度条将会卡住        
   		if(fromUser){
                    imageView.setPivotX(imageView.getWidth()/2);
                    imageView.setPivotY(imageView.getHeight()/2);//支点在图片中心
            		//progress表示毫秒数，非常大，所以转化为比较容易观察的数据
                    imageView.setRotation(progress/30);
                    current_time.setText(time.format(progress));
                    //与服务通信
                    Parcel data = Parcel.obtain();
                    Parcel reply = Parcel.obtain();
                    data.writeInt(progress);
                    try{
                        mBinder.transact(SEEK_CODE, data, reply, 0);
                    }catch (RemoteException e){
                        Log.e("STOP:", "onClick: " + e.toString() );
                    }
                }
    }
    

    @Override
    public void onStartTrackingTouch(SeekBar seekBar) {

    }

    @Override
    public void onStopTrackingTouch(SeekBar seekBar) {

    }
});
```

## Handler的使用

现在已经实现了最基础的播放、暂停、停止、拖动播放功能

但是还需要播放时更新当前进度，更新进度条，旋转封面



因为要一直监听着`MediaPlayer`的进度，然而它又没有`onProgressChangeListener`方法可以用

所以需要单独用一个线程来观察它的变化，然后使用`Handler`通知UI变化

为什么不用主线程，因为它的本质就是一个循环，观察UI线程，抽不出身来观察别的东西了

### 创建新线程

有两种方法，要么直接继承`Thread`类，要么实现`Runnable`方法，只要实现了`run`方法就好了

```java
public Runnable  myThread = new Runnable() {
        @Override
        public void run() {
            Message msg = handler.obtainMessage();
            //待补充
            try{
                //与服务通信
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();
                mBinder.transact(CURRENTDURATION_CODE, data, reply, 0);
                msg.arg1 = reply.readInt();
            }catch (Exception e){
                Log.d("Run", "run: " + e.toString());
                return;
            }
            handler.sendMessage(msg);
        }
    };
```

### 创建Handler

```java
@SuppressLint("HandlerLeak")
    private final Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            switch (msg.what) { // 根据消息类型进行操作
          		//待补充
                default:
                    //播放结束后，模拟点击一次停止按键，置位UI
                     if(msg.arg1 >= total_duration)
                        stop.performClick();
                    seekbar.setProgress(msg.arg1);
                    current_time.setText(time.format(new Date(msg.arg1)));
                    imageView.setPivotX(imageView.getWidth()/2);
                    imageView.setPivotY(imageView.getHeight()/2);//支点在图片中心
                    imageView.setRotation(msg.arg1/30);
                    handler.postDelayed(myThread, 1);
            }
        }
    };
```

> 听说这样子可能会造成内存泄漏，不过我感觉这个应用应该不会有这个问题，如果有的话，尝试去把`Activity`设置为单实例
>
> ```xml
> <activity
>     android:name=".MainActivity"
>     android:launchMode="singleInstance">
> ```



现在就能实现播放时进度条移动，封面旋转以及播放时间更新了！

## 退出播放器

首先要重写`onDestory`方法

虽然Service可以在后台服务，可是也保不准什么时候被系统杀掉，我的手机播放完一首歌就会被系统kill了

如果没有在`onDestory`方法中处理好资源的释放，就会弹出恼人的“**应用已停止**”出错对话框

```java
@Override
public void onDestroy(){
    super.onDestroy();
    handler.removeCallbacks(myThread);
    if(sc != null){
        //解绑
        unbindService(sc);
    }
}
```

现在返回键和主页键都已经结束不了这个播放器了，点击退出按钮要怎么结束呢？

很简单，一句代码就够了

```java
quit.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
       finish();
    }
});
```

> 之前我还在里面手动调用了`onDestory`，其实finish方法之后系统会自动调用`onDestory`销毁当前`Activity`，所以多次调用会出错提示：“`java.lang.IllegalStateException: No activity`”，因为第一遍已经销毁了，第二遍已经不存在这个`Activity`了

## 后台播放

其实这个已经使用了`Service`，程序进入后台之后应该还是会继续播放，然而结果并不是这样子

点击`home`键能实现后台播放

点击`back`键却结束了播放

想了想，应该是系统觉得我的应用太简单，点击返回就可以直接杀掉了，所以得制止这种行为！

**重写点击返回键的方法**

```java
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    if (keyCode==KeyEvent.KEYCODE_BACK){
        moveTaskToBack(true);
        return false;
    }
    return super.onKeyDown(keyCode, event);
}
```

> return true:返回键是回到上一个activity
>
> return false:后者会直接最小化应用，重新进入应用之后首先就会看到你所操作的这个`avtivity`!

## 选择歌曲播放

刚刚已经获取了文件访问权限，现在就可以直接打开文件选择窗口了

这里筛选文件类型为**音频**

```java
select.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
                intent.setType("audio/*");
                intent.addCategory(Intent.CATEGORY_OPENABLE);
                startActivityForResult(intent,1);
            }
        });
```

重写选择文件后的返回事件

```java
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (data != null) {
        try{
            //与服务通信
                Parcel send_data = Parcel.obtain();
                Parcel reply = Parcel.obtain();
                send_data.writeString(data.getData().toString());
                try{
                    mBinder.transact(NEWMUSIC_CODE, send_data, reply, 0);
                }catch (RemoteException e){
                    Log.e("STOP:", "onClick: " + e.toString() );
                }
            //设置信息
                seekbar.setProgress(0);
                seekbar.setMax(ms.mp.getDuration());
                total_time.setText(DateFormat.format(time_format, (long)ms.mp.getDuration()));
                current_time.setText(time.format(new Date(0)));
            //......
            //待补充
        }catch (Exception e){
            Log.d("Open file", "onActivityResult: " + e.toString());
        }

    }
    super.onActivityResult(requestCode, resultCode, data);
}
```

这里要注意的是，重新设置播放源文件的时候需要先`reset`，但是不能`release`

不过也可以`release`，然后`ms.mp = new MediaPlayer();`就好了，虽然充分利用了JAVA的垃圾回收机制，不过总感觉这样不是很稳妥

**在返回事件里还得重新解析音乐的信息**

```java
MediaMetadataRetriever mmr = new MediaMetadataRetriever();
mmr.setDataSource(MainActivity.this,data.getData());
//获取媒体标题
music_title.setText(mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_TITLE));
//获取媒体艺术家
music_singer.setText(mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_ARTIST));
//获取媒体图片（专辑封面）
byte[] picture = mmr.getEmbeddedPicture();
if(picture.length!=0){
    Bitmap bitmap = BitmapFactory.decodeByteArray(picture, 0, picture.length);
    imageView.setImageBitmap(bitmap);
}
//释放
mmr.release();
//自动播放
isPlay = false;
play_pause.performClick();
//不自动播放
//模拟停止
//stop.performClick();
```

> 名称: mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_TITLE))
>
> 专辑: mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_ALBUM))
>
> 歌手: mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_ARTIST))
>
> 码率: mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_BITRATE))
>
> 时长:mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_DURATION))
>
> 类型:mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_MIMETYPE))

## 解决一些细节问题

### 点击停止按钮UI不复位

点击停止按钮之后进度条、当前播放时间和封面都没有复位，但是再次点击播放按钮之后还是会会重新开始

调试的时候`handler.removeCallbacks(myThread);`确实起了作用，而且UI也都复位了，但是正常运行的时候却不行，也许是线程的时间间隔太短吧……



那就再想个办法！



新建一个变量指示当前是否是已经停止的状态（需要通知UI复位）

```java
boolean isStop = false;
```

修改`myThread`

```java
public Runnable  myThread = new Runnable() {
        @Override
        public void run() {
            Message msg = handler.obtainMessage();
            if(isStop){
                msg.what = -1;
                handler.sendMessage(msg);
                return;
            }
            try{
                //与服务通信
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();
                mBinder.transact(CURRENTDURATION_CODE, data, reply, 0);
                msg.arg1 = reply.readInt();
            }catch (Exception e){
                Log.d("Run", "run: " + e.toString());
                return;
            }
            handler.sendMessage(msg);
        }
    };
```

修改`handler`

```java
@SuppressLint("HandlerLeak")
    private final Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            switch (msg.what) { // 根据消息类型进行操作
                case -1:
                    handler.removeCallbacks(myThread);
                    seekbar.setProgress(0);
                    current_time.setText(time.format(0));
                    imageView.setRotation(0);
                    break;
                default:
                    if(msg.arg1 >= total_duration)
                        stop.performClick();
                    seekbar.setProgress(msg.arg1);
                    current_time.setText(time.format(new Date(msg.arg1)));
                    imageView.setPivotX(imageView.getWidth()/2);
                    imageView.setPivotY(imageView.getHeight()/2);//支点在图片中心
                    imageView.setRotation(msg.arg1/30);
                    handler.postDelayed(myThread, 1);
            }
        }
    };

```

修改按钮监听事件

```java
play_pause.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //与服务通信
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();
                try{
                    mBinder.transact(PLAY_CODE, data, reply, 0);
                }catch (RemoteException e){
                    Log.e("STOP:", "onClick: " + e.toString() );
                }
                if(isPlay){
                    isPlay = false;
                    play_pause.setImageResource(R.mipmap.play);
                }
                else {
                    isPlay = true;
                    isStop = false;
                    play_pause.setImageResource(R.mipmap.pause);
                    //开始监听
                    myThread.run();
                }

            }
        });
        stop.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
               isPlay = false;
               isStop = true;
               play_pause.setImageResource(R.mipmap.play);
                seekbar.setProgress(0);
                current_time.setText(time.format(0));
                imageView.setRotation(0);
                //与服务通信
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();
                try{
                    mBinder.transact(STOP_CODE, data, reply, 0);
                }catch (RemoteException e){
                    Log.e("STOP:", "onClick: " + e.toString() );
                }

            }
        });
```

这样子的话

如果按了停止按钮 ->

`isStop`就会置位`true` -> 

`myThread`知道之后就会发送值为-1的`msg.what` -> 

`handler`接收到就会让所有UI复位

然后重新点击按钮的话会重新开启另一个线程`mThread`

原来的线程因为没有调用`handler.postDelayed(myThread, 1);`以后再也不会使用了，应该会被回收掉

<!-- more --> 