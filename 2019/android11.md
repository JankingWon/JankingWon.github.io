---
title: Android手机应用开发（十一） |  Retrofit2 + RxJava2 + OkHttp + Restful应用
urlname: android11
tags:
  - Android
  - Restful
  - Retrofit2
  - RxJava
  - RxJava2
  - OKHTTP
  - Gson
copyright: true
category: Android
abbrlink: 43090
date: 2018-12-18 19:28:51
---

## 注意！是Retrofit2+ RxJava2，而不是Retrofit2+ RxJava！

> 实验目的
>
> 1. 理解`Restful`接口
> 2. 学会使用`Retrofit2`
> 3. 复习使用`RxJava`
> 4. 学会使用`OkHttp`

<!-- more --> 

## 效果

![效果](https://blog.janking.cn/post/android11/效果.gif)

## 添加依赖

```java
implementation 'com.android.support:cardview-v7:28.+'
implementation 'com.android.support:recyclerview-v7:28.0.0'
//JSON转换类
implementation 'com.squareup.retrofit2:converter-gson:2.0.0'
//RxJava和RxAndroid
implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
implementation 'io.reactivex.rxjava2:rxjava:2.2.4'
//okhttp和retrofit
implementation 'com.squareup.okhttp3:okhttp:3.2.0'
implementation 'com.squareup.retrofit2:retrofit:2.0.0'
implementation 'com.squareup.retrofit2:converter-scalars:2.0.0'
//很关键!!!
//retrofit对Rxjava2版本的适配
implementation 'com.squareup.retrofit2:adapter-rxjava2:2.2.0'
```

## 项目结构

![1545134400855](https://blog.janking.cn/post/android11/1545134400855.png)

## 新增一个跳转的ACTIVITY

布局写两个按钮就好了

![1545139511959](https://blog.janking.cn/post/android11/1545139511959.png)

`SwitchActivity.java`

```java
public class SwitchActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_switch);

        Button lab10 = findViewById(R.id.lab10);
        Button lab11 = findViewById(R.id.lab11);
        //跳转到BILIBILIAPI
        lab10.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startActivity(new Intent(SwitchActivity.this, MainActivity.class));
            }
        });
        //跳转到GITHUBAPI
        lab11.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startActivity(new Intent(SwitchActivity.this, RepoActivity.class));
            }
        });
    }
}
```

修改`AndroidManifest.xml`，在`SwitchActivity`中添加`intent-filter`，使其成为主界面

（原来这个`intent-filter`是在`MainActivity`里）

```xml
<activity android:name=".SwitchActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

## 获取`Github`某个用户所有仓库(`repository`)

### 新建`GithubRepoObj.java`

![1545133845596](https://blog.janking.cn/post/android11/1545133845596.png)

这是用来装`HTTP`请求的表明用户每个仓库信息的`Json`数据转换的实体类，已经调用了谷歌的`gson`库，所以只要注解`SerializedName`添加正确，可以实现从`Json`到`Class`的自动转换



解析的格式通过得到上述访问`api`链接观察得到

```java
package com.sysu.janking.httpapi.GITHUB;
import com.google.gson.annotations.SerializedName;
public class GithubRepoObj {
        @SerializedName("id")
        private String id;
        @SerializedName("name")
        private String name;
        @SerializedName("description")
        private String description;
        @SerializedName("open_issues_count")
        private int open_issues_count;
        @SerializedName("has_issues")
        private boolean has_issues;
    
        public String getName() {
            return name;
        }

        public String getId() {
            return id;
        }

        public int getOpen_issues_count() {
            return open_issues_count;
        }

        public String getDescription() {
            return description;
        }

    public boolean isHas_issues() {
        return has_issues;
    }
}
```

### 新建`GitHubService.java`

```java
// 不是Class 而是 interface
public interface GitHubService {
    // 这里的List<Repo>即为最终返回的类型，需要保持一致才可解析
    // 之所以使用一个List包裹是因为该接口返回的最外层是一个数组
    @GET("/users/{user_name}/repos")
    Observable<List<GithubRepoObj>> getRepo(@Path("user_name") String user_name);
}
```

这是一个服务接口，用来请求`GithubAPI`

这里使用了`retrofit2`的注解方法，将参数`user_name`放进`url`，再通过`GET`请求，返回具体的`repository`列表，所以被观察者的对象是是一个`GithubRepoObj`的列表(`List`)

### 新建`repo_recycler_item.xml`

这是显示仓库列表的`RecyclerView`单项布局

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
            <TextView
                android:id="@+id/repo_name"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="项目标题"
                android:layout_marginTop="5dp"
                android:textSize="30sp"
                android:textColor="@android:color/black"
                android:gravity="center_horizontal"/>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="项目ID:    "
                    android:layout_marginTop="5dp"
                    android:textSize="20sp"
                    android:textColor="@android:color/black"/>
                <TextView
                    android:id="@+id/repo_id"
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="项目ID"
                    android:layout_marginTop="5dp"
                    android:textSize="20sp"/>
            </LinearLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="存在问题:    "
                    android:layout_marginTop="5dp"
                    android:textSize="20sp"
                    android:textColor="@android:color/black"/>
                <TextView
                    android:id="@+id/repo_issues_count"
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="存在问题"
                    android:layout_marginTop="5dp"
                    android:textSize="20sp"/>
            </LinearLayout>
            <android.support.v7.widget.CardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_margin="10dp"
                app:contentPadding="8dp"
                app:cardBackgroundColor="@android:color/white">
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:id="@+id/repo_description"
                    android:text="项目描述"
                    android:textSize="20sp"/>
            </android.support.v7.widget.CardView>
        </LinearLayout>
    </android.support.v7.widget.CardView>
</android.support.constraint.ConstraintLayout>
```

### 新建`GithubRepoRecyclerAdapter.java`

用来正确显示用户仓库的列表(`RecyclerViewList`)

```java
package com.sysu.janking.httpapi.GITHUB;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import com.sysu.janking.httpapi.R;
import java.util.ArrayList;

public class GithubRepoRecyclerAdapter extends RecyclerView.Adapter<GithubRepoRecyclerAdapter.ViewHolder> {
    private ArrayList<GithubRepoObj> repos;
    private GithubRepoRecyclerAdapter.OnItemClickListener onItemClickListener;

    public GithubRepoRecyclerAdapter(){
        repos = new ArrayList<>();
    }
    @Override
    public GithubRepoRecyclerAdapter.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)
    {
        // 实例化展示的view
        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.repo_recycler_item, parent, false);
        // 实例化viewholder
        GithubRepoRecyclerAdapter.ViewHolder viewHolder = new GithubRepoRecyclerAdapter.ViewHolder(v);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(final GithubRepoRecyclerAdapter.ViewHolder holder, final int position){
        // 绑定数据
        holder.repo_name.setText(repos.get(position).getName());
        //老生常谈！！！int转String
        holder.repo_issues_count.setText(String.valueOf(repos.get(position).getOpen_issues_count()));
        holder.repo_description.setText(repos.get(position).getDescription());
        holder.repo_id.setText(repos.get(position).getId());
        //listener
        holder.itemView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(final View v) {
                if(onItemClickListener != null) {
                    int pos = holder.getLayoutPosition();
                    onItemClickListener.onItemClick(holder.itemView, pos);
                }
            }
        });
    }
    public void addItem(GithubRepoObj githubRepoObj){
        if(repos != null){
            repos.add(githubRepoObj);
            notifyItemInserted(repos.size() - 1);
        }
    }
    @Override
    public int getItemCount() {
        return repos.size();
    }
   
    //清空RecyclerView列表
    public void reset(){
        int count = repos.size();
        repos.clear();
        notifyItemRangeRemoved(0,count);
    }

    public String getRepoName(int position){
        return repos.get(position).getName();
    }

    public void setOnItemClickListener(GithubRepoRecyclerAdapter.OnItemClickListener listener) {
        this.onItemClickListener = listener;
    }

    public interface OnItemClickListener {
        void onItemClick(View view, int position);
    }


    public static class ViewHolder extends RecyclerView.ViewHolder{
        TextView repo_name, repo_id, repo_description, repo_issues_count;
        public ViewHolder(View itemView) {
            super(itemView);
            repo_name = itemView.findViewById(R.id.repo_name);
            repo_id = itemView.findViewById(R.id.repo_id);
            repo_description = itemView.findViewById(R.id.repo_description);
            repo_issues_count = itemView.findViewById(R.id.repo_issues_count);
        }
    }
}
```

这里有几个容易出错的地方

#### 第一点

给列表中添加了数据之后要**通知改变`UI`**

调用的是这个

`notifyItemInserted(repos.size() - 1);`

也就是增加的用户的下标（`ArrayList`理解为数组的话）

#### 第二点

`reset`方法清空列表的时候，也要记得**通知改变`UI`**

`notifyItemRangeRemoved(0,count);`

#### 第三点

`getOpen_issues_count()`方法返回的是`int`值，表示的是该仓库开放的问题数

如果直接向下面这样调用会出错，因为如果`setText`的参数是`int`值，它会当做是资源号，也就是像`mipmap`，`layout`等文件夹里放的资源一样，但是一般不会找到这个`int`值对应的资源，所以应用会崩溃！

```java
 holder.repo_issues_count.setText(repos.get(position).getOpen_issues_count());
```

所以如果要让`TextView`或者`EditText`显示数字的时候，**应该要转换成`String`**，如下

```
holder.repo_issues_count.setText(String.valueOf(repos.get(position).getOpen_issues_count()));
```

因此，对`RecyclerView`进行数据修改的时候，要记得修改存储数据的容器，**更要记得通知`UI`的改变**！



### 新建`RepoActivity`

#### 布局`activity_repo.xml`

![Screenshot_20181218-193814](https://blog.janking.cn/post/android11/Screenshot_20181218-193814.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".GITHUB.RepoActivity">
    <LinearLayout
        android:id="@+id/second_top_view"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:layout_marginStart="10dp"
        android:layout_marginEnd="10dp"
        android:orientation="horizontal">
        <EditText
            android:id="@+id/second_search_id"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:hint="请输入github用户名"
            />
        <Button
            android:id="@+id/second_search_button"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:text="搜索"
            />
    </LinearLayout>
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:scrollbars="vertical"
        app:layout_constraintTop_toBottomOf="@id/second_top_view">
        <android.support.constraint.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <android.support.v7.widget.RecyclerView
                android:id="@+id/second_recycler_view_list"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                >
            </android.support.v7.widget.RecyclerView>
            <View
                android:layout_width="match_parent"
                android:layout_height="50dp"
                app:layout_constraintTop_toBottomOf="@id/second_recycler_view_list"></View>
        </android.support.constraint.ConstraintLayout>
    </ScrollView>
</android.support.constraint.ConstraintLayout>
```

#### 代码`RepoActivity.java`

```java
package com.sysu.janking.httpapi.GITHUB;

import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import com.sysu.janking.httpapi.R;
import java.net.UnknownHostException;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;
import io.reactivex.android.schedulers.AndroidSchedulers;
import io.reactivex.observers.DisposableObserver;
import io.reactivex.schedulers.Schedulers;
import okhttp3.OkHttpClient;
import retrofit2.Retrofit;
import retrofit2.adapter.rxjava2.RxJava2CallAdapterFactory;
import retrofit2.converter.gson.GsonConverterFactory;

public class RepoActivity extends AppCompatActivity {
    private final String baseURL = "https://api.github.com";
    private GithubRepoRecyclerAdapter githubRecyclerAdapter;
    private RecyclerView recycler_view_list;
    private EditText editText;
    private String username;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_repo);
        //recyclerView的设置
        RecyclerView.LayoutManager mLayoutManager = new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false);
        recycler_view_list = findViewById(R.id.second_recycler_view_list);
        githubRecyclerAdapter = new GithubRepoRecyclerAdapter();
        recycler_view_list.setAdapter(githubRecyclerAdapter);
        recycler_view_list.setLayoutManager(mLayoutManager);
        githubRecyclerAdapter.setOnItemClickListener(new GithubRepoRecyclerAdapter.OnItemClickListener() {
            @Override
            public void onItemClick(View view, int position) {
                //跳转到IssuesActivity
                Intent intent = new Intent(RepoActivity.this, IssuesActivity.class);
                intent.putExtra("username",username);
                intent.putExtra("repo",githubRecyclerAdapter.getRepoName(position));
                startActivity(intent);
            }
        });
        //搜索用户点击事件
        editText = findViewById(R.id.second_search_id);
        Button search = findViewById(R.id.second_search_button);
        search.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                username = editText.getText().toString();
                //清空搜索框
                editText.setText("");
                getGithubRepos();
            }
        });
    }
    //获取用户所有仓库的方法
    public void getGithubRepos(){
        //先声明OkHttpClient，因为retrofit时基于okhttp的，在这可以设置一些超时参数等
        OkHttpClient build = new OkHttpClient.Builder()
                .connectTimeout(10, TimeUnit.SECONDS)
                .readTimeout(30, TimeUnit.SECONDS)
                .writeTimeout(10, TimeUnit.SECONDS)
                .build();
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl(baseURL)
                // 设置json数据解析器
                .addConverterFactory(GsonConverterFactory.create())
                //!!!不是下面这样
                //.addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                //这是RXJAVA2的版本
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .client(build)
                .build();
        DisposableObserver<List<GithubRepoObj>> disposableObserver = new DisposableObserver<List<GithubRepoObj>>() {
            @Override
            public void onNext(List<GithubRepoObj> githubRepoObjs) {
                //清空之前的搜索结果
                githubRecyclerAdapter.reset();

                //用户仓库为空
                if(githubRepoObjs.isEmpty()){
                    Toast.makeText(RepoActivity.this, "该用户不存在任何Repository", Toast.LENGTH_SHORT).show();
                    return;
                }
                //添加到显示结果的列表中
                for(GithubRepoObj g : githubRepoObjs){
                    //过滤掉fork他人的项目
                    if(g.isHas_issues())
                        githubRecyclerAdapter.addItem(g);
                }
            }
            @Override
            public void onComplete() {
                Toast.makeText(RepoActivity.this, "搜索完成", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onError(Throwable e) {
                if(e instanceof UnknownHostException)
                    Toast.makeText(RepoActivity.this, "网络连接失败", Toast.LENGTH_SHORT).show();
                //404错误
                else if(e.toString().equals("retrofit2.adapter.rxjava2.HttpException: HTTP 404 Not Found")){
                    Toast.makeText(RepoActivity.this, "不存在该用户", Toast.LENGTH_SHORT).show();
                } else{
                    Toast.makeText(RepoActivity.this, "未知错误", Toast.LENGTH_SHORT).show();
                }
                e.printStackTrace();
            }

        };
        GitHubService gitHubService = retrofit.create(GitHubService.class);
        gitHubService.getRepo(username)
                .subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                //相当重要，rxjava:2.2.4不是Subscriber而是DisposableObserver
                .subscribe(disposableObserver);
    }

}
```

------

**注意这里使用的是`RxJava2`的版本**，很多教程都是`1.1.0`，这两个版本**区别很大！**

比如`RxJava`的观察者是`Subscriber`，但是`RxJava2`的观察者是`DisposableObserver`

而且`Retrofit`调用`RxJava`的时候用的工厂也不一样

下面是正确的调用方式

```
retrofit.addCallAdapterFactory(RxJava2CallAdapterFactory.create())
```

> 就是这个RxJava2CallAdapterFactory困扰了我好久
>
> 之前是一直提示这个错误
>
> ```
> java.lang.IllegalArgumentException: Unable to create call adapter for io.reactivex.Observable<java.util.List<com.sysu.janking.httpapi.GithubRepoObj>> for method GitHubService.getRepo
> ```
>
> ![error](https://blog.janking.cn/post/android11/error.png)

更多可以查看[RxJava2的介绍](https://blog.csdn.net/aiynmimi/article/details/53382567)

------



## 获取`Github`某个用户某个仓库(`repository`)的所有问题(`issue`)

### 新建`GithubIssueObj.java`

```java
package com.sysu.janking.httpapi.GITHUB;
import com.google.gson.annotations.SerializedName;
public class GithubIssueObj {
    @SerializedName("title")
    private String title;
    @SerializedName("state")
    private String state;
    @SerializedName("created_at")
    private String created_at;
    @SerializedName("body")
    private String body;
    public String getTitle() {
        return title;
    }
    public String getBody() {
        return body;
    }
    public String getCreated_at() {
        return created_at;
    }
    public String getState() {
        return state;
    }
}
```

### 修改`GitHubService.java`

添加一个`GET`请求的方法

返回的是某个用户某个仓库所有`Issue`的集合，参数是`Github`用户名和仓库名

```java
@GET("/repos/{user_name}/{repo}/issues")
Observable<List<GithubIssueObj>> getIssue(@Path("user_name") String user_name, @Path("repo") String repo);
```

### 新建`issue_recycler_item.xml`

这是显示问题(Issue)的`RecyclerView`的单项布局

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
            <TextView
                android:id="@+id/issue_title"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="ISSUE标题"
                android:layout_marginTop="5dp"
                android:textSize="30sp"
                android:textColor="@android:color/black"
                android:gravity="center_horizontal"/>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="创建时间:    "
                    android:layout_marginTop="5dp"
                    android:textSize="20sp"
                    android:textColor="@android:color/black"/>
                <TextView
                    android:id="@+id/issue_create_date"
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="创建时间"
                    android:layout_marginTop="5dp"
                    android:textSize="20sp"/>
            </LinearLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="问题状态:    "
                    android:layout_marginTop="5dp"
                    android:textSize="20sp"
                    android:textColor="@android:color/black"/>
                <TextView
                    android:id="@+id/issue_state"
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="问题状态"
                    android:layout_marginTop="5dp"
                    android:textSize="20sp"/>
            </LinearLayout>
            <android.support.v7.widget.CardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_margin="10dp"
                app:contentPadding="8dp"
                app:cardBackgroundColor="@android:color/white">
                <TextView
                    android:id="@+id/issue_body"
                    android:layout_width="wrap_content"
                    android:layout_height="match_parent"
                    android:text="问题描述"
                    android:layout_marginTop="5dp"
                    android:textSize="20sp"/>
            </android.support.v7.widget.CardView>

        </LinearLayout>
    </android.support.v7.widget.CardView>
</android.support.constraint.ConstraintLayout>
```

### 新建`GithubIssueRecyclerAdapter.java`

```java
package com.sysu.janking.httpapi.GITHUB;

import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import com.sysu.janking.httpapi.R;

import java.util.ArrayList;

public class GithubIssueRecyclerAdapter extends RecyclerView.Adapter<GithubIssueRecyclerAdapter.ViewHolder>{
    private ArrayList<GithubIssueObj> issues;

    public GithubIssueRecyclerAdapter(){
        issues = new ArrayList<>();
    }
    @Override
    public GithubIssueRecyclerAdapter.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)
    {
        // 实例化展示的view
        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.issue_recycler_item, parent, false);
        // 实例化viewholder
        GithubIssueRecyclerAdapter.ViewHolder viewHolder = new GithubIssueRecyclerAdapter.ViewHolder(v);
        return viewHolder;
    }
    @Override
    public void onBindViewHolder(final GithubIssueRecyclerAdapter.ViewHolder holder, final int position){
        // 绑定数据
        holder.issue_body.setText(issues.get(position).getBody());
        holder.issue_create_date.setText(issues.get(position).getCreated_at());
        holder.issue_state.setText(issues.get(position).getState());
        holder.issue_title.setText(issues.get(position).getTitle());
    }
    public void addItem(GithubIssueObj githubIssueObj){
        if(issues != null){
            issues.add(githubIssueObj);
            notifyItemInserted(issues.size() - 1);
        }
    }
    //同样有个清空列表的操作
    public void reset(){
        int count = issues.size();
        issues.clear();
        notifyItemRangeRemoved(0,count);
    }
    @Override
    public int getItemCount() {
        return issues.size();
    }
    public static class ViewHolder extends RecyclerView.ViewHolder{
        TextView issue_title, issue_create_date, issue_body, issue_state;
        public ViewHolder(View itemView) {
            super(itemView);
            issue_title = itemView.findViewById(R.id.issue_title);
            issue_create_date = itemView.findViewById(R.id.issue_create_date);
            issue_body = itemView.findViewById(R.id.issue_body);
            issue_state = itemView.findViewById(R.id.issue_state);
        }
    }
}
```

### 新建`IssuesActivity`

#### 布局`activity_issues.xml`

![Screenshot_20181218-193827](https://blog.janking.cn/post/android11/Screenshot_20181218-193827.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".GITHUB.IssuesActivity">
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:scrollbars="vertical">
        <android.support.constraint.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <LinearLayout
                android:id="@+id/issues_top_view"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginStart="10dp"
                android:layout_marginEnd="10dp"
                android:orientation="vertical">
                <EditText
                    android:id="@+id/issues_title"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:hint="输入Title"
                    />
                <EditText
                    android:id="@+id/issues_body"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:hint="输入Body"
                    />
                <Button
                    android:id="@+id/issues_commit_button"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_horizontal"
                    android:text="增加ISSUE"
                    />
            </LinearLayout>

            <android.support.v7.widget.RecyclerView
                android:id="@+id/issues_recycler_view_list"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                app:layout_constraintTop_toBottomOf="@id/issues_top_view"
                >
            </android.support.v7.widget.RecyclerView>
        </android.support.constraint.ConstraintLayout>
    </ScrollView>
</android.support.constraint.ConstraintLayout>
```

#### 代码`IssuesActivity.java`

```java
package com.sysu.janking.httpapi.GITHUB;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import com.sysu.janking.httpapi.R;
import java.util.List;
import java.util.concurrent.TimeUnit;
import io.reactivex.android.schedulers.AndroidSchedulers;
import io.reactivex.observers.DisposableObserver;
import io.reactivex.schedulers.Schedulers;
import okhttp3.OkHttpClient;
import retrofit2.Retrofit;
import retrofit2.adapter.rxjava2.RxJava2CallAdapterFactory;
import retrofit2.converter.gson.GsonConverterFactory;

public class IssuesActivity extends AppCompatActivity {
    private final String baseURL = "https://api.github.com";
    private RecyclerView recycler_view_list;
    private GithubIssueRecyclerAdapter githubIssueRecyclerAdapter;
    private EditText issues_title, issues_body;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_issues);
        //recyclerView的设置
        RecyclerView.LayoutManager mLayoutManager = new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false);
        recycler_view_list = findViewById(R.id.issues_recycler_view_list);
        githubIssueRecyclerAdapter = new GithubIssueRecyclerAdapter();
        recycler_view_list.setAdapter(githubIssueRecyclerAdapter);
        recycler_view_list.setLayoutManager(mLayoutManager);
        //提交ISSUE
        issues_title = findViewById(R.id.issues_title);
        issues_body = findViewById(R.id.issues_body);
        Button submit = findViewById(R.id.issues_commit_button);
        submit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                postGithubIssue(issues_title.getText().toString(), issues_body.getText().toString());
            	//这个方法后面定义
            }
        });

        //获取ISSUES
        getGithubIssues(getIntent().getStringExtra("username"),
                getIntent().getStringExtra("repo"));

    }
    //获取用户某一个仓库所有问题的方法
    public void getGithubIssues(String username, String repo) {
        //先声明OkHttpClient，因为retrofit时基于okhttp的，在这可以设置一些超时参数等
        OkHttpClient build = new OkHttpClient.Builder()
                .connectTimeout(10, TimeUnit.SECONDS)
                .readTimeout(30, TimeUnit.SECONDS)
                .writeTimeout(10, TimeUnit.SECONDS)
                .build();
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl(baseURL)
                // 设置json数据解析器
                .addConverterFactory(GsonConverterFactory.create())
                //!!!不是下面这样
                //.addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                //这是RXJAVA2的版本
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .client(build)
                .build();
        GitHubService gitHubService = retrofit.create(GitHubService.class);
            DisposableObserver<List<GithubIssueObj>> disposableObserver = new DisposableObserver<List<GithubIssueObj>>() {
                @Override
                public void onNext(List<GithubIssueObj> githubIssueObjs) {
                    //清除掉之前的列表
                    githubIssueRecyclerAdapter.reset();
                    if(githubIssueObjs.isEmpty()){
                        Toast.makeText(IssuesActivity.this, "该仓库不存在任何Issue", Toast.LENGTH_SHORT).show();
                        return;
                    }
                    //添加到显示结果的列表中
                    for(GithubIssueObj g : githubIssueObjs){
                        githubIssueRecyclerAdapter.addItem(g);
                    }
                }
                @Override
                public void onComplete() {
                    Toast.makeText(IssuesActivity.this, "搜索完成", Toast.LENGTH_SHORT).show();
                }
                @Override
                public void onError(Throwable e) {
                    e.printStackTrace();
                }
            };
            gitHubService.getIssue(username, repo)
                    .subscribeOn(Schedulers.newThread())
                    .observeOn(AndroidSchedulers.mainThread())
                    //相当重要，rxjava:2.2.4不是Subscriber而是DisposableObserver
                    .subscribe(disposableObserver);
    }
    //提交issue的方法
    public void postGithubIssue(String title, String body){
        //......
    }
}
```

这里的方法`getGithubIssues`同样也是使用`Rxjava2`来异步请求仓库的所有问题



------



## ![img](https://blog.janking.cn/post/android11/03F7F6F9.png)对`Github`某个仓库(`repository`)添加问题(`issue`)

### API原型

**接口**：`https://api.github.com/repos/{用户名}/{仓库名}/issues`  

**方法**：`POST`  

**提交内容**：

```json
{
  "title": "Creating issue from API",
  "body": "Posting a issue from Insomnia"
}
```



### 添加提交数据的类`GithubIssueJsonObj`

```java
public class GithubIssueJsonObj {
    public String title;
    public String body;
    public GithubIssueJsonObj(String title, String body){
        this.title = title;
        this.body = body;
    }
}
```

这个类很简单，但是作用却不小，因为`POST`需要提交一个`Json`的数据，所以需要通过这个方法把实体类传输过去，应该可以自动解析出来

### 修改`GitHubService.java`

添加一个`POST`请求的方法

```java
@Headers("Authorization: token {YOUR_TOKEN}")//需要补充TOKEN
@POST("/repos/{user_name}/{repo}/issues")
Observable<GithubPostIssueObj> postIssue(@Path("user_name") String user_name,
                                             @Path("repo") String repo,
                                             @Body GithubIssueJsonObj githubIssueObj);
```

因为`GitHub`上发表问题需要登录，所以需要验证身份

但是在这种简单的网络请求中不会通过输入用户名和密码来验证身份（不安全是其中一个原因）

`GitHub`采用了更加安全同时也很简便的办法验证，就是通过授权`Token`

### 新建`GithubPostIssueObj.java`

`POST`方式提交数据之后也会有返回的数据，但是我们不需要利用这个数据，所以我就随便写了个类，随便写了个成员，对结果不会进行分析，只是为了让`Retrofit`正常运行而已

> 因为对于不保留结果不知道怎么处理，肯定会有更好的方法处理这种情况

```java
package com.sysu.janking.httpapi.GITHUB;

import com.google.gson.annotations.SerializedName;

public class GithubPostIssueObj {
    @SerializedName("id")
    private int id;
    public int getTitle() {
        return id;
    }
}
```

### 获取GitHub的Token

在已经登录`GitHub`的账户上点击`Setting`

![1545137613065](https://blog.janking.cn/post/android11/1545137613065.png)



点击`Developer Setting`

![1545137564490](https://blog.janking.cn/post/android11/1545137564490.png)

点击`Personal access tokens`，并点击`Generate new token`

![1545137683072](https://blog.janking.cn/post/android11/1545137683072.png)

应该只要允许`repo`权限就好了（不然权限太大很危险）

![1545137843183](https://blog.janking.cn/post/android11/1545137843183.png)

点击确认之后应该就会返回`Token`列表显示具体的`token`值

> `token`值只会显示一次，所以得及时复制保存，忘了的话，点击上图中的`Regenerate token`就好了

![1545137913662](https://blog.janking.cn/post/android11/1545137913662.png)

然后把这个`token`填写到`GitHubService`里面的`token`项中



### 补充`postGithubIssue`方法

其实跟获取仓库的问题差不多，只不过`GitHubService`调用的方法不一样而已

方法中提交的数据是`new GithubIssueJsonObj(title, body)`



而且需要注意的是，提交成功后会在`onNext`中再次调用`getGithubIssues`更新问题(`Issue`)列表

```java
public void postGithubIssue(String title, String body){
    //先声明OkHttpClient，因为retrofit时基于okhttp的，在这可以设置一些超时参数等
    OkHttpClient build = new OkHttpClient.Builder()
            .connectTimeout(10, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .writeTimeout(10, TimeUnit.SECONDS)
            .build();
    Retrofit retrofit = new Retrofit.Builder()
            .baseUrl(baseURL)
            // 设置json数据解析器
            .addConverterFactory(GsonConverterFactory.create())
            //!!!不是下面这样
            //.addCallAdapterFactory(RxJavaCallAdapterFactory.create())
            //这是RXJAVA2的版本
            .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
            .client(build)
            .build();
    GitHubService gitHubService = retrofit.create(GitHubService.class);
    DisposableObserver<GithubPostIssueObj> disposableObserver = new DisposableObserver<GithubPostIssueObj>() {
        @Override
        public void onNext(GithubPostIssueObj response) {
            //重新显示ISSUES，更新界面
            getGithubIssues(getIntent().getStringExtra("username"),
                    getIntent().getStringExtra("repo"));
        }
        @Override
        public void onComplete() {
            Toast.makeText(IssuesActivity.this, "提交完成", Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onError(Throwable e) {
            Toast.makeText(IssuesActivity.this, "提交失败", Toast.LENGTH_SHORT).show();
            e.printStackTrace();
        }

    };
    gitHubService.postIssue(getIntent().getStringExtra("username"),
            getIntent().getStringExtra("repo"),new GithubIssueJsonObj(title, body))
            .subscribeOn(Schedulers.newThread())
            .observeOn(AndroidSchedulers.mainThread())
            //相当重要，rxjava:2.2.4不是Subscriber而是DisposableObserver
            .subscribe(disposableObserver);
}
```

## 参考链接

[RxJava2的介绍](https://blog.csdn.net/aiynmimi/article/details/53382567)

[Rxjava2和Retrofit的适配](http://www.cnblogs.com/tonycheng93/p/6346466.html)

[Retrofit之请求参数](https://www.jianshu.com/p/162b36a84e8a)

[HTTP请求方法中，如Get/Post方式中的HTTP头 Authorization 怎么添加?](http://www.cnblogs.com/tonycheng93/p/6346466.html)

<!-- more --> 

