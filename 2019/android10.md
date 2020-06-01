---
title: Android手机应用开发（十） | HttpURLConnection的使用以及解析JSON数据
urlname: android10
tags:
  - Android
  - API
  - Bitmap
  - JSON
  - RxJava
  - RecyclerView
  - Handle
copyright: true
category: Android
abbrlink: 53099
date: 2018-12-08 17:01:25
---

>   实验目的
>
> 1. 学会使用HttpURLConnection请求访问Web服务
> 2. 学习Android线程机制，学会线程更新UI
> 3. 学会解析JSON数据
> 4. 学习CardView布局技术

<!-- more --> 

## 效果

![效果图](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android10/gif5.gif)

## 添加依赖

```java
//json解析类
implementation 'com.squareup.retrofit2:converter-gson:2.1.0'
//rxjava类
implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
implementation 'io.reactivex.rxjava2:rxjava:2.2.4'
//recyclerview ui
implementation 'com.android.support:recyclerview-v7:28.0.0'
//cardview ui
implementation'com.android.support:cardview-v7:28.+'
```

> 本例使用的`RxJava`是2.2.4版本，与1.X版本有较大区别！！！

## 添加权限

在`app/src/main/AndroidManifest.xml`中添加

```
<uses-permission android:name="android.permission.INTERNET"/>
```

这次的网络访问权限不用动态申请啦！！！



但是

高版本(`Android 8`以上)的`SDK`可能会报这样的错误

`Cleartext HTTP traffic to xxx not permitted`

> Google表示，为保证用户数据和设备的安全，针对下一代 Android 系统(Android P) 的应用程序，将要求默认使用加密连接，这意味着 Android P 将禁止 App 使用所有未加密的连接，因此运行 Android P 系统的安卓设备无论是接收或者发送流量，未来都不能明码传输

这里采用最简单的办法

在`app/src/main/AndroidManifest.xml`中的`application`中添加

```xml
android:usesCleartextTraffic="true"
```

强行使用明文传输



更多办法：https://stackoverflow.com/questions/45940861/android-8-cleartext-http-traffic-not-permitted

## 使用`HttpURLConnection`获取网络数据

示例`API`网址: https://space.bilibili.com/ajax/top/showTop?mid=2

```java
URL url = new URL(baseURL + "showTop?mid=" + editText.getText().toString());
HttpURLConnection conn = (HttpURLConnection)url.openConnection();
conn.setConnectTimeout(5000);
conn.setRequestMethod("GET");
if(conn.getResponseCode() == 200){
    //InputStream转String
    BufferedInputStream bis = new BufferedInputStream((InputStream)conn.getContent());
    ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
    int result = bis.read();
    while(result != -1) {
        byteArrayOutputStream.write((byte) result);
        result = bis.read();
    }
    //结果是byteArrayOutputStream.toString()
}
```

这样子就可以通过`byteArrayOutputStream.toString()`得到字符串

不过字符串是`JSON`类型数据，需要解析才能方便使用

## `JSON`解析类

观察网页返回的`json`结果

```json
{"status":true,"data":{"aid":349,"state":0,"cover":"http:\/\/i2.hdslb.com\/bfs\/archive\/8d15f47650e6e95e11ad7a3d6ae06ea52231169a.jpg","title":"[\u4eba\u751f\u7684\u5bfc\u5e08\u677e\u5188\u4fee\u9020]\u6771\u65b9\u4fee\u5922\u9020","content":"sm5159640","play":186049,"duration":"01:23","video_review":936,"create":"2009-09-08 17:00:16","rec":"e'm'm'm'm'm","count":1}}
```

根据`json`中的属性新建实体类`RecyclerObj.java`

> 实体类中不要求实现所有的属性

```java
import android.graphics.Bitmap;
import com.google.gson.annotations.SerializedName;
import java.util.ArrayList;
import java.util.List;
public class RecyclerObj {
    @SerializedName("status")
    private Boolean status;
    @SerializedName("data")
    private data idata;
    public static class data {
        @SerializedName("aid")
        private String aid;
        @SerializedName("title")
        private String title;
        @SerializedName("cover")
        private String cover;
        @SerializedName("play")
        private String play;
        @SerializedName("duration")
        private String duration;
        //评论
        @SerializedName("video_review")
        private String video_review;
        @SerializedName("create")
        private String create;
        @SerializedName("content")
        private String content;
        public String getAid() {
            return aid;
        }
        public String getTitle() {
            return title;
        }
        public String getPlay() {
            return play;
        }
        public String getVideo_review() {
            return video_review;
        }
        public String getContent() {
            return content;
        }
        public String getCover() {
            return cover;
        }
        public String getCreate() {
            return create;
        }
        public String getDuration() {
            return duration;
        }
    }
    public Boolean getStatus() {
        return status;
    }
    public data getIdata() {
        return idata;
    }
}    
```



后来就可以使用`new Gson().fromJson(byteArrayOutputStream.toString(), RecyclerObj.class)`将

`String`转换为`RecyclerObj`对象了



还有个实体类`PreviewObj.java`用来获取预览图的链接返回的`JSON`数据

```java
package com.sysu.janking.httpapi;

import com.google.gson.annotations.SerializedName;

import java.util.ArrayList;

public class PreviewObj {
    @SerializedName("code")
    private int code;
    @SerializedName("data")
    private data idata;
    public static class data {
        @SerializedName("img_x_len")
        private int img_x_len;
        @SerializedName("img_y_len")
        private int img_y_len;
        @SerializedName("img_x_size")
        private int img_x_size;
        @SerializedName("img_y_size")
        private int img_y_size;
        @SerializedName("image")
        private ArrayList<String> image;
        @SerializedName("index")
        private ArrayList<Integer> index;

        public ArrayList<String> getImage() {
            return image;
        }

        public int getImg_x_len() {
            return img_x_len;
        }

        public int getImg_x_size() {
            return img_x_size;
        }

        public int getImg_y_len() {
            return img_y_len;
        }

        public int getImg_y_size() {
            return img_y_size;
        }

        public ArrayList<Integer> getIndex() {
            return index;
        }
    }

    public data getIdata() {
        return idata;
    }

    public int getCode() {
        return code;
    }
}
```

## 结合`RxJava`

但是像上面一样直接运行的话会报`NetworkOnMainThreadException`的异常，因为网络访问不能在主线程中进行，容易造成阻塞或者UI瘫痪

在`MainActivity.java`中添加被观察者

```java
private CompositeDisposable mCompositeDisposable = new CompositeDisposable();
private Observable<RecyclerObj> observable = Observable.create(new ObservableOnSubscribe<RecyclerObj>() {
    @Override
    public void subscribe(ObservableEmitter<RecyclerObj> observableEmitter) throws Exception {
        URL url = new URL(baseURL + "showTop?mid=" + editText.getText().toString());
        HttpURLConnection conn = (HttpURLConnection)url.openConnection();
        conn.setConnectTimeout(5000);
        conn.setRequestMethod("GET");
        //InputStream转String
        if(conn.getResponseCode() == 200){
            BufferedInputStream bis = new BufferedInputStream((InputStream)conn.getContent());
            ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
            int result = bis.read();
            while(result != -1) {
                byteArrayOutputStream.write((byte) result);
                result = bis.read();
            }
            observableEmitter.onNext(new Gson().fromJson(byteArrayOutputStream.toString(), RecyclerObj.class));
        }
        observableEmitter.onComplete();
    }
});
```

在搜索按钮的点击事件中添加观察者

```java
DisposableObserver<RecyclerObj> disposableObserver = new DisposableObserver<RecyclerObj>() {
    @Override
    public void onNext(RecyclerObj recyclerObj) {
        if(recyclerObj.getStatus()){
            //添加到显示结果的列表中
            recyclerAdapter.addItem(recyclerObj);
        }
        else//这句理论上一直不会执行
            Toast.makeText(MainActivity.this, "数据库中不存在记录", Toast.LENGTH_SHORT).show();
    }
    @Override
    public void onComplete() {
        Toast.makeText(MainActivity.this, "搜索完成", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onError(Throwable e) {
        if(e instanceof UnknownHostException)
            Toast.makeText(MainActivity.this, "网络连接失败", Toast.LENGTH_SHORT).show();
        else if(e instanceof com.google.gson.JsonSyntaxException){
            Toast.makeText(MainActivity.this, "数据库中不存在记录", Toast.LENGTH_SHORT).show();
        }else{
            Toast.makeText(MainActivity.this, "未知错误", Toast.LENGTH_SHORT).show();
        }
        Log.d("onError", "onError: " + e.toString());
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
```

其实如果`JSON`返回的不一定就是上述定义好的结构

如输入一个不存在的用户名，就会出现以下另一种结构的JSON数据

```json
{"status":false,"data":"\u53c2\u6570\u9519\u8bef"}
```

跟上述的区别在于，

错误返回`data`字段只有一个`String`类型的数据

而正确返回`data`字段是另一个`JSON`数据（对应一个实体类）

所以当`gson`解析`JSON`时在`data`后面找不到`{`，便会抛出`com.google.gson.JsonSyntaxException`异常

所以可以通过判断这个异常间接判断用户名是否存在数据



> 而通过`recyclerObj.getStatus()`这个结果返回`false`判断用户名不存在的语句理论上一直不回执行，因为如果用户名不存在的话一定没有`data`字段的`JSON`数据，所以一定会抛出异常，不能成功解析出`status`字段



## 通过`RecyclerView`显示结果

### 布局文件`recycler_item.xml`

![1544334288002](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android10/1544334288002.png)

布局中使用了`cardview`，上面是一个`ImageView`（用来显示视频封面）和一个`Progressbar`（显示加载进度），但是此时`ImageView`是隐藏的（`visibility = “gone”`)，

因为进度条和封面不能同时显示，**当封面加载完成后进度条会隐藏**

中间有一个`SeekBar`，通过拖动显示视频的预览，但是默认是不可拖动的（在`java`文件里设置），**只有预览图加载成功后才可以拖动**

下面是一些`TextView`显示简介、播放数、评论数、时长、创建事件等

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginBottom="30dp"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <android.support.v7.widget.CardView
        app:cardCornerRadius="8dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:contentPadding="5dp"
        android:layout_margin="10dp"
        app:cardBackgroundColor="@color/cardview_shadow_start_color"
        android:orientation="vertical">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">
            <ImageView
                android:id="@+id/result_cover"
                android:layout_width="match_parent"
                android:src="@mipmap/ic_launcher"
                android:visibility="gone"
                android:adjustViewBounds="true"
                android:layout_height="wrap_content" />
            <ProgressBar
                android:id="@+id/progress_bar"
                android:visibility="visible"
                android:layout_gravity="center"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
            <SeekBar
                android:id="@+id/seek_bar"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="5dp"
                />
            <TextView
                android:id="@+id/result_title"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="这是标题"
                android:layout_marginTop="5dp"
                android:textSize="20sp"
                android:textColor="@android:color/black"
                android:gravity="center_horizontal"/>
            <android.support.v7.widget.CardView
                app:cardCornerRadius="8dp"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:layout_margin="10dp"
                app:layout_constraintTop_toBottomOf="@id/top_view">
                <TextView
                    android:id="@+id/result_content"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:gravity="center_horizontal"
                    android:layout_marginBottom="5dp"
                    android:textSize="15sp"
                    android:textColor="@android:color/black"
                    android:text="简介"/>
            </android.support.v7.widget.CardView>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:gravity="center_horizontal">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:layout_marginEnd="5dp"
                    android:text="播放:"/>
                <TextView
                    android:id="@+id/result_play"
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="111"
                    android:layout_marginEnd="20dp"
                    />
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:layout_marginEnd="5dp"
                    android:text="评论:"/>
                <TextView
                    android:id="@+id/result_review"
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="111"
                    android:layout_marginEnd="20dp"
                    />
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:layout_marginEnd="5dp"
                    android:text="时长:"/>
                <TextView
                    android:id="@+id/result_duration"
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="111"
                    />
            </LinearLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="5dp"
                android:orientation="horizontal"
                android:gravity="center_horizontal">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:gravity="center_horizontal"
                    android:layout_marginEnd="10dp"
                    android:text="创建于 "/>
                <TextView
                    android:id="@+id/result_create"
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:gravity="center_horizontal"
                    android:text="2018-01-01"/>
            </LinearLayout>
        </LinearLayout>

    </android.support.v7.widget.CardView>
</android.support.constraint.ConstraintLayout>
```

### 实体类`RecyclerObj.java`

直接复用`Json`实体类就好了就好了

### 适配器`RecyclerAdapter`

```java
public class RecyclerAdapter extends RecyclerView.Adapter<RecyclerAdapter.ViewHolder> {

    private ArrayList<RecyclerObj> mData;
    private Context context;
    private String baseURL = "https://api.bilibili.com/";
    public static final int GET_DATA_SUCCESS = 1;
    public static final int NETWORK_ERROR = 2;
    public static final int SERVER_ERROR = 3;

    public RecyclerAdapter(Context context){
        mData = new ArrayList<>();
        this.context = context;
    }
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)
    {
        // 实例化展示的view
        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.recycler_item, parent, false);
        // 实例化viewholder
        ViewHolder viewHolder = new ViewHolder(v);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(final ViewHolder holder, final int position) {
       //设置图片宽度占满屏幕
       //... 
        
        // 绑定数据
        holder.result_title.setText(mData.get(position).getIdata().getTitle());
        holder.result_content.setText(mData.get(position).getIdata().getContent());
        holder.result_create.setText(mData.get(position).getIdata().getCreate());
        holder.result_duration.setText(mData.get(position).getIdata().getDuration());
        holder.result_play.setText(mData.get(position).getIdata().getPlay());
        holder.result_review.setText(mData.get(position).getIdata().getVideo_review());
        //通过时间字符串得到毫秒数
       	//...
        
       
        //当加载完成时，设置拖动条可拖动
        final Handler handler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                switch (msg.what) {
                    case GET_DATA_SUCCESS:
                        //设置封面
                        holder.result_cover.setImageBitmap(holder.result_bitmap);
                        //隐藏进度条
                        holder.result_progressBar.setVisibility(View.GONE);
                        //显示封面
                        holder.result_cover.setVisibility(View.VISIBLE);
                        //设置拖动条可拖动
                        //防止缺失预览图报异常
                        if(!holder.preview_bitmaps.isEmpty())
                            holder.seekBar.setEnabled(true);
                        break;
                }

            }
        };
        //RXJAVA
        //获取预览图的网页JSON数据
        CompositeDisposable mCompositeDisposable = new CompositeDisposable();
        Observable<PreviewObj> observableGetImageUrl = Observable.create(new ObservableOnSubscribe<PreviewObj>() {
            @Override
            public void subscribe(ObservableEmitter<PreviewObj> observableEmitter) throws Exception {
                URL url = new URL(baseURL + "pvideo?aid=" + mData.get(position).getIdata().getAid());
                HttpURLConnection conn = (HttpURLConnection)url.openConnection();
                conn.setConnectTimeout(5000);
                conn.setRequestMethod("GET");
                //将得到的数据转为字符串，最后解析json为PreviewObj
                if(conn.getResponseCode() == 200){
                    BufferedInputStream bis = new BufferedInputStream((InputStream)conn.getContent());
                    ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
                    int result = bis.read();
                    while(result != -1) {
                        byteArrayOutputStream.write((byte) result);
                        result = bis.read();
                    }
                    observableEmitter.onNext(new Gson().fromJson(byteArrayOutputStream.toString(), PreviewObj.class));
                }
                observableEmitter.onComplete();
            }
        });
        DisposableObserver<PreviewObj> disposableObserver = new DisposableObserver<PreviewObj>() {
            @Override
            public void onNext(final PreviewObj previewObj) {
                //新线程获取图片
                new Thread() {
                    @Override
                    public void run() {
                        try {
                            //获取封面的图片
                            holder.result_bitmap = getBitmap(mData.get(position).getIdata().getCover());
                            //获取预览图的的图片，并解析为bitmap保存到数组里
                            holder.preview_bitmaps = cutBitmap(
                                    getBitmap(previewObj.getIdata().getImage().get(0)),
                                    previewObj.getIdata().getImg_x_len(),
                                    previewObj.getIdata().getIndex().size() - 2,
                                    previewObj.getIdata().getImg_x_size(),
                                    previewObj.getIdata().getImg_y_size());
                        }catch (NullPointerException e){
                            Log.d("NULLE", "onNext: " + e.toString());
                        }
                        catch (IOException e){
                            Log.d("IOE", "onNext: " + e.toString());
                        }finally {
                            //利用Message把图片发给Handler
                            Message msg = Message.obtain();
                            msg.what = GET_DATA_SUCCESS;
                            handler.sendMessage(msg);
                        }
                    }
                }.start();
                Log.d ("Adapter: Next", "");
            }
            @Override
            public void onComplete() {
                Log.d ("Adapter: Complete", "");
            }

            @Override
            public void onError(Throwable e) {
                Log.d ("Adapter: Next", e.toString());
            }
        };
        //在新线程监听
        observableGetImageUrl.subscribeOn(Schedulers.newThread()).observeOn(AndroidSchedulers.mainThread()).subscribe(disposableObserver);
        //管理DisposableObserver的容器
        mCompositeDisposable.add(disposableObserver);
		
        //SeekBar的拖动事件
        //...
        
        
        //提示
        Toast.makeText(context, "加载完成", Toast.LENGTH_SHORT).show();
    }
    @Override
    public int getItemCount()
    {
        return mData.size();
    }

    public void addItem(RecyclerObj recyclerObj){
        if(mData != null){
            mData.add(recyclerObj);
            notifyItemInserted(mData.size() - 1);
        }
    }

    public static class ViewHolder extends RecyclerView.ViewHolder {
        TextView  result_title, result_duration, result_review, result_create, result_play, result_content;
        ImageView result_cover;
        ProgressBar result_progressBar;
        SeekBar seekBar;
        //保存封面
        Bitmap result_bitmap;
        //保存预览图
        ArrayList<Bitmap> preview_bitmaps;
        public ViewHolder(View itemView) {
            super(itemView);
            result_cover = itemView.findViewById(R.id.result_cover);
            result_title = itemView.findViewById(R.id.result_title);
            result_content = itemView.findViewById(R.id.result_content);
            result_create = itemView.findViewById(R.id.result_create);
            result_play = itemView.findViewById(R.id.result_play);
            result_duration = itemView.findViewById(R.id.result_duration);
            result_review = itemView.findViewById(R.id.result_review);
            result_progressBar = itemView.findViewById(R.id.progress_bar);
            seekBar = itemView.findViewById(R.id.seek_bar);
            preview_bitmaps = new ArrayList<>();
            //默认不可拖动
            seekBar.setEnabled(false);
        }
    }
    //通过url得到bitmap文件
    public static Bitmap getBitmap(String path) throws IOException, ProtocolException {
        //...
    }
    //裁剪bitmap图片
    public static ArrayList<Bitmap> cutBitmap(Bitmap bitmap, int img_x_len, int total_len, int img_x_size, int img_y_size){
        //...
    }

}
```

`Adapter`确实干了不少事！

首先对于`RecyclerView`里的每个项设置内容，然后对于数据中的`aid`，

**在`RxJava`中**用另一个"https://api.bilibili.com/"`api`链接获取预览图的`JSON`数据

​	得到数据之后， 在`RxJava`中**另开一个线程**得到`JSON`中的`URL`指向的图片，转换为`bitmap`，并存储到`ArrayList`中；而且还通过`RecyclerObj`中的`cover`指向的`URL`得到封面图存储到`bitmap`变量中



下面是对`Adapter`的补充

#### 设置图片宽度占满屏幕

```java
//获取屏幕宽度
int screenWidth = context.getResources().getDisplayMetrics().widthPixels;
ViewGroup.LayoutParams lp = holder.result_cover.getLayoutParams();
//宽度为屏幕宽度
lp.width = screenWidth;
//高度自适应
lp.height = LinearLayout.LayoutParams.WRAP_CONTENT;
holder.result_cover.setLayoutParams(lp);
//最大允许宽度和高度
holder.result_cover.setMaxWidth(screenWidth);
holder.result_cover.setMaxHeight(screenWidth);
```

不过要在`xml`里设置`ImageView`的`android:adjustViewBounds="true"`

#### 通过时间字符串得到毫秒数

```java
String total = mData.get(position).getIdata().getDuration();
Date start = new Date();
try{
   date = new SimpleDateFormat("mm:ss").parse(total);
   start = new SimpleDateFormat("mm:ss").parse("00:00");
}catch (ParseException e){
    Log.d("getMs", "onBindViewHolder: " + e.toString());
}
//设置拖动条(总数为秒数)
holder.seekBar.setMax((int)(date.getTime() - start.getTime()) / 1000);
holder.seekBar.setProgress(0);
```

`new SimpleDateFormat("mm:ss").parse(total)`得到的结果是缺省`1970年1月1日00:00:00`

如输入字符串是`"01:23"`，指定格式为`"mm:ss"`

得到的结果是

`Thu Jan 01 00:01:23 GMT+08:00 1970`

因此要得到需要的时间的毫秒数（没有得到秒数的方法……）只有减去另一个基准值如`"00:00"`

#### `SeekBar`的拖动事件

```java
holder.seekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
    @Override
    public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
        holder.result_cover.setImageBitmap(holder.preview_bitmaps.get( progress * (holder.preview_bitmaps.size()-1) / holder.seekBar.getMax()));
        Log.d("test", "onProgressChanged: " +  progress * (holder.preview_bitmaps.size()-1) / holder.seekBar.getMax());
    }

    @Override
    public void onStartTrackingTouch(SeekBar seekBar) {
        holder.result_cover.setImageBitmap(holder.result_bitmap);
    }

    @Override
    public void onStopTrackingTouch(SeekBar seekBar) {
        seekBar.setProgress(0);
        holder.result_cover.setImageBitmap(holder.result_bitmap);
    }
});
```

因为没有解析出每张预览图的时间点，所以就粗略的把预览图当做整个视频上平均分配的时间点，通过这句把一定时长上的`SeekBar`值转换为一定数量预览图的某一张

`progress * (holder.preview_bitmaps.size()-1) / holder.seekBar.getMax()`



#### 通过`url`得到`bitmap`文件

```java
//通过url得到bitmap文件
public static Bitmap getBitmap(String path) throws IOException, ProtocolException {
    URL url = new URL(path);
    HttpURLConnection conn = (HttpURLConnection)url.openConnection();
    conn.setConnectTimeout(5000);
    conn.setRequestMethod("GET");
    if(conn.getResponseCode() == 200){
        InputStream inputStream = conn.getInputStream();
        Bitmap bitmap = BitmapFactory.decodeStream(inputStream);
        inputStream.close();
        return bitmap;
    }
    return null;
}
```

#### 裁剪`bitmap`图片

```java
    public static ArrayList<Bitmap> cutBitmap(Bitmap bitmap, int img_x_len, int total_len, int img_x_size, int img_y_size){
        ArrayList<Bitmap> bitmaps = new ArrayList<>();
        //竖直方向取图，total_len / img_x_len + 1, 比如有27张图片，每行10张，那么就有3行
        //x表示横坐标（从左上角水平方向算）
        for(int i = 0; i < total_len / img_x_len + 1 && i < 10; i++){
            //水平方向取图，水平方向即每行有img_x_len张图片
            //y表示纵坐标（从左上角竖直方向算）
            for(int j = 0; j < img_x_len && i * img_x_len + j < total_len; j++){
                //基于原图，取正方形左上角x坐标
                int retX = img_x_size * j;
                int retY = img_y_size * i;
                bitmaps.add(Bitmap.createBitmap(bitmap, retX, retY, img_x_size, img_y_size, null, false));
            }
        }
        return bitmaps;
    }
```

### `MainActivity.java`中绑定`Adapter`

```java
//about RecyclerView
RecyclerView.LayoutManager mLayoutManager = new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false);
recycler_view_list = findViewById(R.id.recycler_view_list);
recyclerAdapter = new RecyclerAdapter(this);
recycler_view_list.setAdapter(recyclerAdapter);
recycler_view_list.setLayoutManager(mLayoutManager);
```

<!-- more --> 