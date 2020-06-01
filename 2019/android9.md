---
title: Android手机应用开发（九） | RxJava（RxAndroid）的简单使用
urlname: android9
tags:
  - Android
  - RxJava
  - RxAndroid
copyright: true
category: Android
abbrlink: 51391
date: 2018-12-03 18:59:52
---

> 实验目的
>
> 1. 学会使用MediaPlayer
> 2. 学习RxJava，使用RxJava更新UI
> 3. 学会使用Service进行后台工作
> 4. 学会使用Service与Activity进行通信

<!-- more --> 

## 效果

![gif5新文件](https://blog.janking.cn/post/android8/gif5新文件.gif)

## RxJava简介

GitHub地址：

[ReactiveX团队](https://github.com/ReactiveX)

[RxJava](https://github.com/ReactiveX/RxJava )

[RxAndroid](https://github.com/ReactiveX/RxAndroid )

`RxJava`是主体，其实还有`RxAndroid`、`RxGo`、`RxPY`、`RxSwift`等适配

这里使用的是`RxAndroid`和`RxJava`

> RxAndroid并不包括全部的RxJava，而是侧重Android的特性进行添加，所以防止缺少依赖库，还是都得使用

## 其它代码环境

这里使用的是[Android手机应用开发（八） | 制作简单音乐播放器](https://blog.janking.cn/post/android8.html) 的大部分代码

只是将更新UI的操作从`Handler`更改为`RxJava`



### 注册服务`AndroidManifest.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="wang.janking.mymusic">
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <application
    ...
        <service
            android:name=".MusicService"
            android:exported="true" />
    ...
    </application>

</manifest>
```

### 新建服务`MusicService.java`

```java
package wang.janking.mymusic;

import android.app.Service;
import android.content.Intent;
import android.media.MediaPlayer;
import android.net.Uri;
import android.os.Binder;
import android.os.Environment;
import android.os.IBinder;
import android.os.Parcel;
import android.os.RemoteException;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
import android.util.Log;
import android.widget.Toast;

import java.io.IOException;

public class MusicService extends Service {
    public final IBinder binder = new MyBinder();
    private MediaPlayer mp =  new MediaPlayer();
    private int isFinish = -1;
    private final int PLAY_CODE = 1, STOP_CODE = 2, SEEK_CODE = 3, NEWMUSIC_CODE = 4, CURRENTDURATION_CODE = 5, TOTALDURATION = 6;
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        try {
            mp.setDataSource(Environment.getExternalStorageDirectory() + "/data/山高水长.mp3");
            mp.prepare();
            mp.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
                @Override
                public void onCompletion(MediaPlayer mp) {
                    isFinish = 1;
                }
            });
        } catch (IOException e) {
            Log.e("prepare error", "getService: " + e.toString());
        }
        return binder;
    }


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
                    reply.writeInt(isFinish);
                    break;
            }
            return super.onTransact(code, data, reply, flags);
        }
    }
    public void play_pause() {
        if (mp.isPlaying()) {
            mp.pause();
        } else {
            mp.start();
            isFinish = -1;
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
    @Override
    public void onDestroy() {
        super.onDestroy();
        if(mp!= null){
            mp.reset();
            mp.release();
        }
    }

}
```

### 布局文件 `activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <de.hdodenhof.circleimageview.CircleImageView
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/profile_image"
        android:layout_width="match_parent"
        android:layout_height="290dp"
        android:layout_marginTop="30dp"
        android:src="@drawable/img"
        app:layout_constraintTop_toTopOf="parent" />
    <TextView
        android:id="@+id/music_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="山高水长"
        android:layout_marginTop="10dp"
        android:textColor="@android:color/black"
        app:layout_constraintTop_toBottomOf="@id/profile_image"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <TextView
        android:id="@+id/music_singer"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="中山大学合唱团"
        android:layout_marginTop="3dp"
        app:layout_constraintTop_toBottomOf="@id/music_title"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/playing_status"
        android:orientation="horizontal"
        android:layout_marginStart="10dp"
        android:layout_marginEnd="10dp"
        android:layout_marginTop="10dp"
        app:layout_constraintTop_toBottomOf="@id/music_singer">
        <TextView
            android:id="@+id/current_time"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="00:00"/>

        <SeekBar
            android:id="@+id/seekbar"
            style="?android:attr/progressBarStyleHorizontal"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_marginEnd="5dp"
            android:layout_marginStart="5dp"
            android:layout_weight="1" />
        <TextView
            android:id="@+id/total_time"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="99:99"/>
    </LinearLayout>
    <ImageButton
        android:id="@+id/select_music"
        android:src="@mipmap/file"
        android:padding="0dp"
        app:layout_constraintTop_toBottomOf="@id/playing_status"
        android:layout_marginTop="20dp"
        android:layout_marginStart="20dp"
        android:scaleType="fitXY"
        app:layout_constraintStart_toStartOf="parent"
        android:layout_width="40dp"
        android:layout_height="40dp"/>
    <LinearLayout
        app:layout_constraintTop_toBottomOf="@id/playing_status"
        app:layout_constraintStart_toEndOf="@id/select_music"
        android:layout_marginTop="30dp"
        app:layout_constraintEnd_toStartOf="@id/quit"
        android:layout_width="wrap_content"
        android:orientation="horizontal"
        android:layout_height="wrap_content">
        <ImageButton
            android:id="@+id/play_pause"
            android:src="@mipmap/play"
            android:padding="0dp"
            android:scaleType="fitStart"
            android:layout_width="50dp"
            android:layout_marginEnd="30dp"
            android:layout_height="50dp" />
        <ImageButton
            android:id="@+id/stop"
            android:src="@mipmap/stop"
            android:padding="0dp"
            android:scaleType="fitXY"
            android:layout_width="50dp"
            android:layout_height="50dp" />
    </LinearLayout>
    <ImageButton
        android:id="@+id/quit"
        android:src="@mipmap/back"
        android:padding="0dp"
        android:layout_marginTop="20dp"
        app:layout_constraintEnd_toEndOf="parent"
        android:scaleType="fitXY"
        android:layout_marginEnd="20dp"
        app:layout_constraintTop_toBottomOf="@id/playing_status"
        android:layout_width="40dp"
        android:layout_height="40dp" />
</android.support.constraint.ConstraintLayout>
```

### 代码文件

```java
package wang.janking.mymusic;

import android.Manifest;


import android.content.ComponentName;
import android.content.Intent;
import android.content.ServiceConnection;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.media.MediaMetadataRetriever;
import android.os.IBinder;
import android.os.Parcel;
import android.os.RemoteException;
import android.support.annotation.NonNull;
import android.support.v4.app.ActivityCompat;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.text.format.DateFormat;
import android.util.Log;
import android.view.KeyEvent;
import android.view.View;
import android.widget.ImageButton;
import android.widget.ImageView;
import android.widget.SeekBar;
import android.widget.TextView;

import org.reactivestreams.Subscriber;

import java.text.SimpleDateFormat;
import java.util.Date;

import io.reactivex.Observable;
import io.reactivex.ObservableEmitter;
import io.reactivex.ObservableOnSubscribe;
import io.reactivex.Observer;
import io.reactivex.android.schedulers.AndroidSchedulers;
import io.reactivex.disposables.CompositeDisposable;
import io.reactivex.observers.DisposableObserver;
import io.reactivex.schedulers.Schedulers;

public class MainActivity extends AppCompatActivity {
    //
    private IBinder mBinder;
    private final int PLAY_CODE = 1, STOP_CODE = 2, SEEK_CODE = 3, NEWMUSIC_CODE = 4, CURRENTDURATION_CODE = 5, TOTALDURATION = 6;
    private int total_duration = 0;
    //读写权限
    private static String[] PERMISSIONS_STORAGE = {
            Manifest.permission.READ_EXTERNAL_STORAGE,
            Manifest.permission.WRITE_EXTERNAL_STORAGE};
    //请求状态码
    private static int REQUEST_PERMISSION_CODE = 1;
    ImageView imageView;
    SeekBar seekbar;
    ImageButton play_pause, stop, select, quit;
    TextView current_time, total_time, music_title, music_singer;
    boolean isPlay = false;
    boolean isStop = false;
    private SimpleDateFormat time = new SimpleDateFormat("mm:ss");
    private String time_format = "mm:ss";
	
    
    private ServiceConnection sc = new ServiceConnection() {
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

        @Override
        public void onServiceDisconnected(ComponentName name) {
        }
    };
    /*public Runnable  myThread = new Runnable() {
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
    };*/
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ActivityCompat.requestPermissions(this, PERMISSIONS_STORAGE, REQUEST_PERMISSION_CODE);
        play_pause = findViewById(R.id.play_pause);
        stop = findViewById(R.id.stop);
        select = findViewById(R.id.select_music);
        quit = findViewById(R.id.quit);
        seekbar = findViewById(R.id.seekbar);
        imageView = findViewById(R.id.profile_image);
        current_time = findViewById(R.id.current_time);
        total_time = findViewById(R.id.total_time);
        music_singer = findViewById(R.id.music_singer);
        music_title = findViewById(R.id.music_title);
        imageView.setPivotX(imageView.getWidth()/2);
        imageView.setPivotY(imageView.getHeight()/2);//支点在图片中心
        //设置一些UI
        setSomething();
    }
    void setSomething(){
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
                    //订阅观察者
                   //......
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

        seekbar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                if(fromUser){
                    imageView.setPivotX(imageView.getWidth()/2);
                    imageView.setPivotY(imageView.getHeight()/2);//支点在图片中心
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

        select.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
                intent.setType("audio/*");
                intent.addCategory(Intent.CATEGORY_OPENABLE);
                startActivityForResult(intent,1);
            }
        });

        quit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                finish();
            }
        });
    }
    @Override
    public void onDestroy(){
        super.onDestroy();
        if(sc != null){
            unbindService(sc);
        }
    }
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == REQUEST_PERMISSION_CODE) {
            Intent intent = new Intent(this, MusicService.class);
            bindService(intent, sc, BIND_AUTO_CREATE);
        }
    }
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
                total_duration = reply.readInt();
                seekbar.setProgress(0);
                seekbar.setMax(total_duration);
                total_time.setText(DateFormat.format(time_format, total_duration));
                current_time.setText(time.format(new Date(0)));
                //设置歌曲信息
                MediaMetadataRetriever mmr = new MediaMetadataRetriever();
                mmr.setDataSource(MainActivity.this,data.getData());
                music_title.setText(mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_TITLE));
                music_singer.setText(mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_ARTIST));
                byte[] picture = mmr.getEmbeddedPicture();
                if(picture.length!=0){
                    Bitmap bitmap = BitmapFactory.decodeByteArray(picture, 0, picture.length);
                    imageView.setImageBitmap(bitmap);
                }
                mmr.release();
                //自动播放
				isPlay = false;
				play_pause.performClick();
				//不自动播放
				//模拟停止
				//stop.performClick();
            }catch (Exception e){
                Log.d("Open file", "onActivityResult: " + e.toString());
            }

        }
        super.onActivityResult(requestCode, resultCode, data);
    }
    //点击返回键
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode==KeyEvent.KEYCODE_BACK){
            moveTaskToBack(true);
            return false;
        }
        return super.onKeyDown(keyCode, event);
    }
}
```



## 添加依赖

有两种办法

- 直接在`Build.gradle（Module app)`里添加

  ```xml
  //这是截止到目前的最新版
  implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
  implementation 'io.reactivex.rxjava2:rxjava:2.2.4'
  ```

- 下载`Jar`包手动添加依赖

  这个我试过，但是从一些小网站上下载的缺少很多东西，而且很多jar包在CSDN上下载要积分！！！知识怎么能收费呢！


但是直接添加依赖代码的方法可能会出现下载失败的情况，因为要经过谷歌的库……

这样子需要更改`Build.gradle（Module Project)`

```xml
// Top-level build file where you can add configuration options common to all sub-projects/modules.
buildscript {
    repositories {
        // maven库
        def cn = "http://maven.aliyun.com/nexus/content/groups/public/"
        def abroad = "http://central.maven.org/maven2/"
        // 先从url中下载jar若没有找到，则在artifactUrls中寻找
        maven {
            url cn
            artifactUrls abroad
        }
        maven { url "http://maven.aliyun.com/nexus/content/repositories/jcenter"}
        // 保留google源
        google()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.4'


        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
allprojects {
    repositories {
        // maven库
        def cn = "http://maven.aliyun.com/nexus/content/groups/public/"
        def abroad = "http://central.maven.org/maven2/"
        // 先从url中下载jar若没有找到，则在artifactUrls中寻找
        maven {
            url cn
            artifactUrls abroad
        }
        maven { url "http://maven.aliyun.com/nexus/content/repositories/jcenter"}
        // 保留google源
        google()
    }
}
task clean(type: Delete) {
    delete rootProject.buildDir
}
```

## 添加被观察者（Observable）

现在新的版本不是简单添加一个`Observable`就好，而是需要寄放在`CompositeDisposable`里面

所以在`MainActivity.java`中添加两个成员变量

```java
//RxJAVA变量
private CompositeDisposable mCompositeDisposable = new CompositeDisposable();
private Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> observableEmitter) throws Exception {
            while (true) {
                //与服务通信
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();
                try {
                    //获取信息
                    mBinder.transact(CURRENTDURATION_CODE, data, reply, 0);
                    //每隔一毫秒查询一次歌曲的状态
                    Thread.sleep(1);
                } catch (Exception exception){
                    Log.d("SERVICE CONNECTION", "onServiceConnected: " + exception.toString());
                }
                //读取当前进度
                observableEmitter.onNext(reply.readInt());
                //reply.readInt() == 1 读取 isFinish判断歌曲是否被动停止
                // isStop 判断歌曲是否主动停止
                if(reply.readInt() == 1 || isStop)
                    break;
            }
            observableEmitter.onComplete();
    }
});
```

这里其实做的主要操作就是每隔一毫秒读取歌曲的进度，然后调用`onNext()`让观察者更新UI，如果歌曲已经停止就调用`onComplete()`结束观察过程



> 因为我想实现歌曲播放完毕自动回到起点，同时UI置位，所以这里用了很多变量（如，`isFinish`，`isStop`等）
>
> 但是我知道**这些变量有冗余**，没有精简，但是功能是没问题的！

## 添加观察者（DisposableObserver）

现在的版本不是叫`Observer`了，而是加了个`Disposable`，表示可处理，就把它当做对事件的具体处理来理解就好了

更改`MainActivity.java`播放按钮的监听事件

```java
//观察者
play_pause.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
               //...
                if(isPlay){
                    //...
                }
                else {
                    //...
                    //订阅观察者
                    DisposableObserver<Integer> disposableObserver = new DisposableObserver<Integer>() {
                        @Override
                        //设置UI
                        public void onNext(Integer integer) {
                            Log.d("onNext", "" + integer);
                            seekbar.setProgress(integer);
                            current_time.setText(time.format(new Date(integer)));
                            imageView.setPivotX(imageView.getWidth()/2);
                            imageView.setPivotY(imageView.getHeight()/2);//支点在图片中心
                            imageView.setRotation(integer/30);
                        }
						//模拟点击停止按键
                        @Override
                        public void onComplete() {
                            stop.performClick();
                        }

                        @Override
                        public void onError(Throwable e) {
                            Log.d("onError", "" + e.toString());
                        }
                    };
                    //在新线程监听
                    observable.subscribeOn(Schedulers.newThread())
                        //在主线程更新
                        .observeOn(AndroidSchedulers.mainThread())
                        //绑定
                        .subscribe(disposableObserver);
                    //管理DisposableObserver的容器
                    mCompositeDisposable.add(disposableObserver);
                }

            }
        });
```

为什么这个观察者变量不像被观察者一样作为一个成员变量呢？

**因为它们只能订阅一次！**

这里是每次点击播放按钮就开始播放，**并且开始监听UI改变**，然后歌曲播放完毕（或者点击停止按钮）就调用`onComplete()`方法，那么这对观察者和被观察者的生命也就终止了……

但是重新播放或者选择新的歌曲的话会报错

```
12-03 19:59:08.238 28919-28919/wang.janking.mymusic E/AndroidRuntime: FATAL EXCEPTION: main
    Process: wang.janking.mymusic, PID: 28919
    io.reactivex.exceptions.ProtocolViolationException: It is not allowed to subscribe with a(n) wang.janking.mymusic.MainActivity$1 multiple times. Please create a fresh instance of wang.janking.mymusic.MainActivity$1 and subscribe that to the target source instead.
```

所以要在每次需要监听给的时候动态创建一个**局部变量**`disposableObserver`

以后再调用的时候就又是一个新的变量了

## 销毁

如果`Activity`要被销毁时，我们的后台任务没有执行完，那么就会导致`Activity`不能正常回收，而对于每一个`Observer`，都会有一个`Disposable`对象用于管理，而`RxJava`提供了一个`CompositeDisposable`类用于管理这些`Disposable`，我们只需要将其将入到该集合当中，在`Activity`的`onDestroy`方法中，调用它的`clear`方法，就能避免内存泄漏的发生。

修改`onDestroy`方法

```java
@Override
public void onDestroy(){
    super.onDestroy();
    //清除所有的观察者
    mCompositeDisposable.clear();
    if(sc != null){
        unbindService(sc);
    }
}
```

<!-- more --> 