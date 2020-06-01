---
title: Android手机应用开发（七） | 数据存储（下）
urlname: android7
tags:
  - Android
  - SQLite
  - Bitmap
  - ListView
  - ContentProvider
  - 获取通讯录
  - 获取图片
copyright: true
category: Android
abbrlink: 18904
date: 2018-11-13 22:02:11
---

> 实验目的
>
> 1. 学习SQLite数据库的使用。
> 2. 学习ContentProvider的使用。
> 3. 复习Android界面编程。



这次大概是做一个有登录、注册、评论、点赞等功能的小型APP

<!-- more --> 

效果如下:（图片比较大）

![GIF](https://blog.janking.cn/post/android7/GIF-1542127554823.gif)

## 登录注册页面的切换

![1](https://blog.janking.cn/post/android7/1.gif)

两个按钮用`RadioButton`就可以实现了

```xml
<RadioGroup
    android:id="@+id/radio_group"
    android:layout_marginTop="120dp"
    app:layout_constraintTop_toTopOf="parent"
    android:layout_width="match_parent"
    android:layout_height="40dp"
    android:orientation="horizontal">
    <RadioButton
        android:id="@+id/login_button"
        android:text="Login"
        android:checked="true"
        android:layout_width="wrap_content"
        android:layout_height="match_parent" />

    <RadioButton
        android:id="@+id/register_button"
        android:text="Register"
        android:layout_width="wrap_content"
        android:layout_height="match_parent" />
</RadioGroup>
```

**但是**，如何实现两种情况下显示不同布局，并且两个按钮都刚好在填写的信息底下呢

这里可以用设置控件的`Visibility`实现的！



我把按钮上方的所有东西放在一个`View`里面（也就是登录界面的两个输入框，注册页面的三个输入框，一个头像显示框）

通过`ConstraintLayout`的特点让两个按钮固定在这个大的View里面，登录状态或者注册状态让头像框和确认密框隐藏就好了

> 其实还有另一种办法：
>
> 只需要用四个控件（**复用两个输入框**）
>
> **登录页面的头像框隐藏，确认密码栏隐藏， 然后在注册页面时显示**
>
> 但是会有挺多问题（感觉是），因为注册和登录用同样的输入框会让后期事件处理以及显示`HINT`的操作有点繁琐
>



这就是那个大的`View`，包含了6个控件


```xml
<android.support.constraint.ConstraintLayout
    android:id="@+id/edit_area"
    app:layout_constraintTop_toBottomOf="@id/radio_group"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <EditText
        android:id="@+id/login_username"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:hint="Username"/>
    <EditText
        android:id="@+id/login_password"
        android:layout_width="match_parent"
        android:inputType="textPassword"
        android:layout_height="40dp"
        android:hint="Password"
        app:layout_constraintTop_toBottomOf="@id/login_username"/>

    <ImageButton
        android:visibility="gone"
        android:src="@mipmap/add"
        android:id="@+id/register_image"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:scaleType="fitXY"
        android:padding="0dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
    <EditText
        android:visibility="gone"
        android:id="@+id/register_username"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:hint="Username"
        app:layout_constraintTop_toBottomOf="@id/register_image"/>
    <EditText
        android:visibility="gone"
        android:inputType="textPassword"
        android:id="@+id/register_password"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:hint="New Password"
        app:layout_constraintTop_toBottomOf="@id/register_username"/>
    <EditText
        android:visibility="gone"
        android:id="@+id/register_confirm_password"
        android:inputType="textPassword"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:hint="Confirm Password"
        app:layout_constraintTop_toBottomOf="@id/register_password"/>
</android.support.constraint.ConstraintLayout>
```

这是底下的两个按钮

```xml
<Button
    android:id="@+id/button_ok"
    android:layout_width="90dp"
    android:layout_height="50dp"
    android:text="OK"
    app:layout_constraintTop_toBottomOf="@id/edit_area"/>
<Button
    android:id="@+id/button_clear"
    android:layout_width="90dp"
    android:layout_height="50dp"
    android:text="CLEAR"
    android:layout_marginStart="30dp"
    app:layout_constraintStart_toEndOf="@id/button_ok"
    app:layout_constraintTop_toBottomOf="@id/edit_area"/>
```

然后在`JAVA`代码里切换`VISIBILITY`属性

```java
final RadioGroup radioGroup = findViewById(R.id.radio_group);
radioGroup.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
    @Override
    public void onCheckedChanged(RadioGroup group, int checkedId) {
        if(((RadioButton)findViewById(checkedId)).getText().toString().equals("Login")){
            register_confirm_password.setVisibility(View.GONE);
            register_image.setVisibility(View.GONE);
            register_password.setVisibility(View.GONE);
            register_username.setVisibility(View.GONE);

            login_username.setVisibility(View.VISIBLE);
            login_password.setVisibility(View.VISIBLE);
            status = true;
        }else{
            register_confirm_password.setVisibility(View.VISIBLE);
            register_image.setVisibility(View.VISIBLE);
            register_password.setVisibility(View.VISIBLE);
            register_username.setVisibility(View.VISIBLE);

            login_username.setVisibility(View.GONE);
            login_password.setVisibility(View.GONE);
            status = false;
        }
    }
});
```

> 注意：这里设置为`GONE`而不是`INVISIBLE`，因为`INVISIBLE`虽然不显示但是占用空间，而`GONE`不会占用空间

## SQLite的简单使用

这里创建一个`SQLiteHelper`用来存储用户名，密码，和头像

```java
package com.example.myapplication;
import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

import java.util.ArrayList;

public class UserSQLiteHelper extends SQLiteOpenHelper {
    private static final String DB_NAME= "db_project3";
    private static final String TABLE_NAME = "user";
    private static final int DB_VERSION = 1;

    private static final String[] COLUMNS = {"username","password","image"};

    public UserSQLiteHelper(Context context) {
        super(context, DB_NAME, null, DB_VERSION);
    }
    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        String CREATE_TABLE = "CREATE TABLE if not exists "
                + TABLE_NAME
                + " ( username TEXT PRIMARY KEY, password TEXT, image TEXT)";
        sqLiteDatabase.execSQL(CREATE_TABLE);
    }
    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int ii) {

    }
    //添加操作
    public Boolean addUser(String name, String password, String image) {
        SQLiteDatabase db = getWritableDatabase();
        Cursor cursor =
                db.query(TABLE_NAME, // a. table
                        COLUMNS, // b. column names
                        " username = ?", // c. selections
                        new String[] { name }, // d. selections args
                        null, // e. group by
                        null, // f. having
                        null, // g. order by
                        null); // h. limit

        // 3. if we got results get the first one
        if (cursor.moveToFirst()){
            db.close();
            return false;
        }
        ContentValues cv = new ContentValues();
        cv.put("username", name);
        cv.put("password", password);
        cv.put("image", image);
        db.insert(TABLE_NAME, null, cv);
        db.close();
        return true;
    }
    //验证登录
    public int loginVerify(String name, String password){
        SQLiteDatabase db = getWritableDatabase();
        Cursor cursor =
                db.query(TABLE_NAME, // a. table
                        COLUMNS, // b. column names
                        " username = ?", // c. selections
                        new String[] { name }, // d. selections args
                        null, // e. group by
                        null, // f. having
                        null, // g. order by
                        null); // h. limit

        if (!cursor.moveToFirst()){
            db.close();
            return 1;//username not found
        }

        if(!password.equals(cursor.getString(1))){
            db.close();
            return 2;//user password not correct
        }
        db.close();
        return 0;
    }
    //得到用户头像
    public String getImage(String name){
        SQLiteDatabase db = getWritableDatabase();
        Cursor cursor =
                db.query(TABLE_NAME, // a. table
                        COLUMNS, // b. column names
                        " username = ?", // c. selections
                        new String[] { name }, // d. selections args
                        null, // e. group by
                        null, // f. having
                        null, // g. order by
                        null); // h. limit

        if (!cursor.moveToFirst()){
            db.close();
            return null;//username not found
        }else {
            String image =  cursor.getString(2);
            cursor.close();
            db.close();
            return  image;
        }
    }
	//删除用户表
    public void dropUser(){
//删除表的SQL语句
        String sql ="DROP TABLE user";
//执行SQL
        this.getWritableDatabase().execSQL(sql);
    }
}
```

在`Activity`的`onCreate`方法中new一个`SQLiteHelper`的实例

```java
userSQLiteHelper = new UserSQLiteHelper(this);
```

然后在注册页面的`button_ok`的监听事件里调用添加用户的方法就实现了注册功能

```java
if(userSQLiteHelper.addUser(register_username.getText().toString(), register_password.getText().toString(), default_image)){
    Toast.makeText(MainActivity.this, "Register Successfully.",Toast.LENGTH_SHORT).show();
    return;
```

在登录页面的button_ok的监听事件里调用验证用户的方法就实现了登录功能

```java
switch (userSQLiteHelper.loginVerify(login_username.getText().toString(), login_password.getText().toString())){
    case 0:
        //跳转到评论页面
        Toast.makeText(MainActivity.this, "Login Successfully.",Toast.LENGTH_SHORT).show();
        Intent intent = new Intent(MainActivity.this, CommentActivity.class);
        intent.putExtra("username", login_username.getText().toString());
        intent.putExtra("image",userSQLiteHelper.getImage(login_username.getText().toString()));
        startActivity(intent);
        break;
    case 1:
        Toast.makeText(MainActivity.this, "Username not existed.",Toast.LENGTH_SHORT).show();
        break;
    case 2:
        Toast.makeText(MainActivity.this, "Invalid Password.",Toast.LENGTH_SHORT).show();
        break;
}
```

## 调用系统相册选择图片



![2](https://blog.janking.cn/post/android7/2.gif)

这里的用户头像其实是一个`ImageButton`，设置监听事件为

```java
String default_image = "android.resource://com.example.myapplication/" + R.mipmap.me;

register_image = findViewById(R.id.register_image);
register_image.setImageURI(Uri.parse(default_image));
register_image.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Intent intent = new Intent();
        intent.setAction(Intent.ACTION_PICK);
        intent.setType("image/*");
        startActivityForResult(intent, 0);
    }
});
```

这样通过调用`Intent`打开了系统的图片查看器，然后还是用`onActivityResult`方法接收点击的图片



## SQLite中保存图片

`Android`数据库中存取图片通常使用两种方式

- 保存图片所在路径

- 将图片以二进制的形式存储（`sqlite3`支持`BLOB`数据类型）

从我之前的代码中可以看出我是通过存储图片的`Uri`存储头像，以后显示该头像就用该`Uri`做参数就好了



> `Android`中的`URI`和`Uri`是不一样的，从`import`的包就可以看出来
>
> import java.net.URI;
> import android.net.Uri;
>
>

```java
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (data != null) {
        // 得到图片的全路径
        Uri uri = data.getData();
        //图片预览
        this.register_image.setImageURI(uri);
        //保存该URI
        default_image = image_file_uri.toString();
    }
    super.onActivityResult(requestCode, resultCode, data);
}
```

> 注：这里的`default_image`保存的是默认用户头像，如果选择了自定义的用户头像它也会改成新的头像，因为最后添加头像的时候是调用它



**但是！！！这样是有问题的**

就是刚添加的时候图片能成功显示，但是关闭APP重新启动或者放在后台又返回的话**图片就显示不了了**

会出现如下警告

```
W/ImageView: Unable to open content: content://com.google.android.apps.photos.contentprovider/-1/1/content%3A%2F%2Fmedia%2Fexternal%2Fimages%2Fmedia%2F6217/NO_TRANSFORM/1633586863

             java.lang.SecurityException: Permission Denial: opening provider com.google.android.apps.photos.contentprovider.MediaContentProvider from ProcessRecord{2e6acd5 26947:com.example.myapplication/u0a67} (pid=26947, uid=10067) that is not exported from uid 10030
```

可以看到是因为没有权限访问这个图片，应该是打开选择图片窗口的`Intent`是一次性的，即使存储了它的路径，以后想得到它操作系统也不允许，为了解决这个问题，而且又不想学习`BLOB`的使用，最简单的办法

**把图片存到自己的地盘！**

就是使用`InternalStorage`把图片复制一份到本应用的`files`文件夹中：[参考教程](https://blog.janking.cn/post/android6.html)

所以修改`onActivityResult`

```java
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (data != null) {
        // 得到图片的全路径
        Uri uri = data.getData();
        ContentResolver cr = this.getContentResolver();
        //存储副本
        try {
            Bitmap bitmap = BitmapFactory.decodeStream(cr.openInputStream(uri));
            //通过UUID生成字符串文件名
            String image_name = UUID.randomUUID().toString() + ".jpg";
            //存储图片
            FileOutputStream out = openFileOutput(image_name, MODE_PRIVATE);
            bitmap.compress(Bitmap.CompressFormat.JPEG, 100, out);
            //获取复制后文件的uri
            Uri image_file_uri = Uri.fromFile(getFileStreamPath(image_name));
            //图片预览
            this.register_image.setImageURI(uri);
            //保存该URI
            default_image = image_file_uri.toString();
            out.flush();
            out.close();
        } catch (FileNotFoundException e) {
            Log.e("FileNotFoundException", e.getMessage(),e);
        } catch (IOException e) {
            Log.w("IOException", e.getMessage(), e);
        }
    }
    super.onActivityResult(requestCode, resultCode, data);
}
```

以后再也不用担心别人不要我读取这个文件了，因为这个文件已经属于我自己了！

## SQLite存储评论

想了又想，还是把评论新建一个表吧，不然每个用户发表很多评论，每个评论又有很多人点赞，还要点赞别人的评论……（想想就恐怖）



修改`UserSQLiteHelper`

这里我用的是**评论发送时间**作为该表的主键，因为时间精确到秒，理论上不会冲突

```java
package com.example.myapplication;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

import java.util.ArrayList;

public class UserSQLiteHelper extends SQLiteOpenHelper {
    private static final String DB_NAME= "db_project3";
    private static final String TABLE_NAME = "user";
    private static final String TABLE_COMMENT_NAME = "comment";
    private static final int DB_VERSION = 1;

    private static final String[] COLUMNS = {"username","password","image"};

    public UserSQLiteHelper(Context context) {
        super(context, DB_NAME, null, DB_VERSION);
    }
    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        String CREATE_TABLE = "CREATE TABLE if not exists "
                + TABLE_NAME
                + " ( username TEXT PRIMARY KEY, password TEXT, image TEXT)";
        sqLiteDatabase.execSQL(CREATE_TABLE);
        String CREATE_COMMENT = "CREATE TABLE if not exists "
                + TABLE_COMMENT_NAME
                + " ( date TEXT PRIMARY KEY, username TEXT, comment TEXT, image TEXT, praise_users TEXT)";
        sqLiteDatabase.execSQL(CREATE_COMMENT);
    }
    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int ii) {

    }
    public Boolean addUser(String name, String password, String image) {
        SQLiteDatabase db = getWritableDatabase();
        Cursor cursor =
                db.query(TABLE_NAME, // a. table
                        COLUMNS, // b. column names
                        " username = ?", // c. selections
                        new String[] { name }, // d. selections args
                        null, // e. group by
                        null, // f. having
                        null, // g. order by
                        null); // h. limit

        // 3. if we got results get the first one
        if (cursor.moveToFirst()){
            db.close();
            return false;
        }
        ContentValues cv = new ContentValues();
        cv.put("username", name);
        cv.put("password", password);
        cv.put("image", image);
        db.insert(TABLE_NAME, null, cv);
        db.close();
        return true;
    }
    public int loginVerify(String name, String password){
        SQLiteDatabase db = getWritableDatabase();
        Cursor cursor =
                db.query(TABLE_NAME, // a. table
                        COLUMNS, // b. column names
                        " username = ?", // c. selections
                        new String[] { name }, // d. selections args
                        null, // e. group by
                        null, // f. having
                        null, // g. order by
                        null); // h. limit

        if (!cursor.moveToFirst()){
            db.close();
            return 1;//username not found
        }

        if(!password.equals(cursor.getString(1))){
            db.close();
            return 2;//user password not correct
        }
        db.close();
        return 0;
    }
    public String getImage(String name){
        SQLiteDatabase db = getWritableDatabase();
        Cursor cursor =
                db.query(TABLE_NAME, // a. table
                        COLUMNS, // b. column names
                        " username = ?", // c. selections
                        new String[] { name }, // d. selections args
                        null, // e. group by
                        null, // f. having
                        null, // g. order by
                        null); // h. limit

        if (!cursor.moveToFirst()){
            db.close();
            return null;//username not found
        }else {
            String image =  cursor.getString(2);
            cursor.close();
            db.close();
            return  image;
        }
    }

    public void addComment(CommentInfo commentInfo){
        SQLiteDatabase db = getWritableDatabase();
        ContentValues cv = new ContentValues();
        cv.put("date", commentInfo.getDate());
        cv.put("username", commentInfo.getUsername());
        cv.put("comment", commentInfo.getComment());
        cv.put("image", commentInfo.getImage());
        cv.put("praise_users", new Convert().array2String(commentInfo.getPraise_users()));
        db.insert(TABLE_COMMENT_NAME, null, cv);
        db.close();
    }

    public void removeComment(CommentInfo commentInfo){
        SQLiteDatabase db = getWritableDatabase();
        String whereClause = "date = ?";
        String[] whereArgs = new String[] { commentInfo.getDate() };
        db.delete(TABLE_COMMENT_NAME, whereClause, whereArgs);
        db.close();
    }

    public void updateComment(String date, ArrayList<String> new_users){
        SQLiteDatabase db = getWritableDatabase();
        ContentValues cv = new ContentValues();
        cv.put("praise_users", new Convert().array2String(new_users));
        String whereClause = "date = ?";
        String[] whereArgs = new String[] { date };
        db.update(TABLE_COMMENT_NAME, cv, whereClause, whereArgs );
        db.close();
    }

    public ArrayList<CommentInfo> getAllComments(){
        ArrayList<CommentInfo> arrayList = new ArrayList<>();

        String query = "SELECT  * FROM " + TABLE_COMMENT_NAME;
        SQLiteDatabase db = this.getWritableDatabase();
        Cursor cursor = db.rawQuery(query, null);

        if (cursor.moveToFirst()) {
            do {
                arrayList.add(new CommentInfo(
                        cursor.getString(0),
                        cursor.getString(1),
                        cursor.getString(2),
                        cursor.getString(3),
                        new Convert().string2Array( cursor.getString(4))));
            } while (cursor.moveToNext());
        }
        cursor.close();
        db.close();
        return arrayList;
    }

    public void dropUser(){
//删除表的SQL语句
        String sql ="DROP TABLE user";
//执行SQL
        this.getWritableDatabase().execSQL(sql);
    }
    public void dropComment(){
//删除表的SQL语句
        String sql ="DROP TABLE comment";
//执行SQL
        this.getWritableDatabase().execSQL(sql);
    }
}

```



还要新建一个存储评论的实体类`CommentInfo.java`

```java
package com.example.myapplication;

import android.text.method.SingleLineTransformationMethod;

import java.util.ArrayList;

public class CommentInfo {
    private String username, date, comment, image;
    private int praise_count;
    private ArrayList<String> praise_users;
    public CommentInfo(String date,String username, String comment, String image, ArrayList<String> praise_users){
        this.username = username;
        this.date = date;
        this.comment = comment;
        this.image = image;
        this.praise_count = praise_users.size();
        this.praise_users = praise_users;
    }

    public String getComment() {
        return comment;
    }

    public String getDate() {
        return date;
    }

    public String getImage() {
        return image;
    }

    public int getPraise_count() {
        return praise_count;
    }

    public String getUsername() {
        return username;
    }

    public void setImage(String image) {
        this.image = image;
    }

    public void setComment(String comment) {
        this.comment = comment;
    }

    public void setDate(String date) {
        this.date = date;
    }

    public void setPraise_count(int praise_count) {
        this.praise_count = praise_count;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPraise_users(ArrayList<String> praise_users) {
        this.praise_users = praise_users;
    }

    public ArrayList<String> getPraise_users() {
        return praise_users;
    }
    //点赞图标点击事件的具体处理
    public Boolean praise_count_change(String current_user){
        //如果该用户之前已经点赞
        for(String u : getPraise_users()){
            if(u.equals(current_user)){
                praise_users.remove(u);
                praise_count--;
                return false;
            }
        }
        //如果该用户之前没有点赞过
        praise_users.add(current_user);
        praise_count++;
        return true;
    }
}
```



这其中涉及到评论点赞的问题，一个评论可能会有很多用户点赞，如何记录下这些用户，因为用户数不是固定的，甚至会无限增长，于是想到把存储点赞用户的数组合并到一个列里，以后再提取出来

最简单的办法就是使用**分隔符分隔用户名**

新建一个转换类`Convert.java`

```java
package com.example.myapplication;
import java.util.ArrayList;
import java.util.Collections;
//用$分割
public class Convert {
    public ArrayList<String> string2Array(String s){
        if(s.isEmpty())
            return new ArrayList<>();
        //下面这句相当关键，不是String[] strings = s.split("$");
        String[] strings = s.split("[$]");
        ArrayList<String> list = new ArrayList(strings.length);
        Collections.addAll(list, strings);
        return list;
    }
    public String array2String(ArrayList<String> arrayList){
        StringBuffer stringBuffer = new StringBuffer();
        for(String i : arrayList){
            stringBuffer.append(i);
            stringBuffer.append("$");
        }
        return  stringBuffer.toString();
    }

}
```

## ListView的使用

效果：

![3](https://blog.janking.cn/post/android7/3-1542127162596.gif)

### Item项布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="80dp"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:descendantFocusability="blocksDescendants"
    android:layout_margin="10dp">
    <ImageView
        android:id="@+id/item_image"
        android:layout_width="40sp"
        android:layout_height="40sp"
        android:src="@mipmap/add"/>
    <TextView
        android:id="@+id/item_username"
        android:layout_width="wrap_content"
        android:layout_height="25dp"
        android:layout_marginStart="5dp"
        android:textSize="20sp"
        android:text="1234567890"
        app:layout_constraintStart_toEndOf="@id/item_image"/>
    <TextView
        android:id="@+id/item_date"
        android:layout_width="wrap_content"
        android:layout_height="15dp"
        android:text="qwertyuiop"
        android:textSize="10sp"
        app:layout_constraintTop_toBottomOf="@id/item_username"
        app:layout_constraintStart_toStartOf="@id/item_username"/>
    <TextView
        android:id="@+id/item_praise_count"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="10"
        android:textSize="15sp"
        android:layout_marginEnd="2dp"
        app:layout_constraintBottom_toBottomOf="@id/item_praise_icon"
        app:layout_constraintEnd_toStartOf="@id/item_praise_icon"
        />
    <ImageButton
        android:id="@+id/item_praise_icon"
        android:layout_width="15dp"
        android:layout_height="15dp"
        android:src="@mipmap/red"
        android:scaleType="fitCenter"
        android:padding="0dp"
        android:layout_marginEnd="5dp"
        app:layout_constraintTop_toTopOf="@id/item_username"
        app:layout_constraintBottom_toBottomOf="@id/item_date"
        app:layout_constraintEnd_toEndOf="parent"/>
    
    <TextView
        android:id="@+id/item_comment"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        app:layout_constraintStart_toStartOf="parent"
        android:text="zxcvbnm"
        android:layout_marginStart="5dp"
        android:textSize="25sp"
        android:textColor="@android:color/black"
        app:layout_constraintTop_toBottomOf="@id/item_image"/>

</android.support.constraint.ConstraintLayout>
```

这个里面很关键的一句

```xml
android:descendantFocusability="blocksDescendants"
```

是为了后面点击事件的处理，**这个属性值表示子有子的焦点，父有父的焦点**

如果没有这个的话，整个`Item`项和内部的`Button`将不会同时可以点击



> 属性`android:descendantFocusability`的值有三种：
>
> `beforeDescendants`：`viewgroup`会优先其子类控件而获取到焦点
> `afterDescendants`：`viewgroup`只有当其子类控件不需要获取焦点时才获取焦点
> `blocksDescendants`：`viewgroup`会覆盖子类控件而直接获得焦点，也就是各有各的焦点
>





### 评论页面布局

直接贴代码吧，毕竟很简单

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".CommentActivity">
    <ListView
    android:layout_marginStart="10dp"
    android:layout_marginEnd="10dp"
    android:id="@+id/comment_list"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_weight="1"
    >
</ListView>
    <View
        android:layout_width="match_parent"
        android:layout_height="2dp"
        android:layout_marginEnd="5dp"
        android:layout_marginStart="5dp"
        android:background="@android:color/black"/>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_margin="5dp">
        <EditText
            android:id="@+id/comment_edit_text"
            android:layout_width="100dp"
            android:layout_weight="1"
            android:layout_height="match_parent"/>
        <Button
            android:text="SEND"
            android:id="@+id/comment_submit"
            android:layout_width="wrap_content"
            android:layout_height="match_parent" />
    </LinearLayout>

</LinearLayout>
```

### Adapter的使用



> 有这三种常用的Adapter
>
> `ArrayAdapter`：支持泛型操作，最简单的一个`Adapter`，只能展现一行文字
>
> `SimpleAdapter`：同样具有良好扩展性的一个`Adapter`，可以自定义多种效果
>
> `BaseAdapter`：抽象类，实际开发中我们会继承这个类并且重写相关方法，用得最多的一个`Adapter`
>



因为`ListView`中设计的元素比较多，还是自己继承`BaseAdapter`写一个`Adapter`吧，

新建`CommentAdapter.java`

```java
package com.example.myapplication;

import android.content.Context;
import android.icu.text.IDNA;
import android.net.Uri;
import android.support.annotation.NonNull;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageButton;
import android.widget.ImageView;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class CommentAdapter extends BaseAdapter{
    private ArrayList<CommentInfo> data;
    private String current_user;
    private Context mContext = null;
    private UserSQLiteHelper userSQLiteHelper;


    public CommentAdapter(Context context, String current_user)
    {
        userSQLiteHelper = new UserSQLiteHelper(context);
        data= userSQLiteHelper.getAllComments();
        mContext = context;
        this.current_user= current_user;
    }

    @Override
    public int getCount()
    {
        int count = 0;
        if (data != null)
        {
            count = data.size();
        }
        return count;
    }

    @Override
    public CommentInfo getItem(int position)
    {
        CommentInfo item = null;

        if (null != data)
        {
            item = data.get(position);
        }

        return item;
    }

    public void removeItem(int position){

        userSQLiteHelper.removeComment(data.get(position));
        if(data!=null && !data.isEmpty()){
            data.remove(position);
            notifyDataSetChanged();
        }

    }

    public void addItem(CommentInfo commentInfo){
        if(data != null && commentInfo!= null){
            data.add(commentInfo);
            notifyDataSetChanged();
        }
        userSQLiteHelper.addComment(commentInfo);
    }

    @Override
    public long getItemId(int position) {
        return 0;
    }

    @Override
    public View getView(final int position, View convertView, ViewGroup parent)
    {
        ViewHolder viewHolder = null;
        if (null == convertView)
        {
            viewHolder = new ViewHolder();
            LayoutInflater mInflater = LayoutInflater.from(mContext);
            convertView = mInflater.inflate(R.layout.comment_item, null);

            viewHolder.item_image= convertView.findViewById(R.id.item_image);
            viewHolder.item_username= convertView.findViewById(R.id.item_username);
            viewHolder.item_date= convertView.findViewById(R.id.item_date);
            viewHolder.item_comment = convertView.findViewById(R.id.item_comment);
            viewHolder.item_praise_count = convertView.findViewById(R.id.item_praise_count);
            viewHolder.praise_button = convertView.findViewById(R.id.item_praise_icon);
            convertView.setTag(viewHolder);
        }
        else
        {
            viewHolder = (ViewHolder) convertView.getTag();
        }

        // set item values to the viewHolder:

        final CommentInfo commentInfo = getItem(position);
        if (null != commentInfo)
        {
            viewHolder.item_image.setImageURI(Uri.parse(commentInfo.getImage()));
            viewHolder.item_username.setText(commentInfo.getUsername());
            viewHolder.item_date.setText(commentInfo.getDate());
            viewHolder.item_comment.setText(commentInfo.getComment());
            viewHolder.item_praise_count.setText(String.valueOf(commentInfo.getPraise_count()));

            //实现点赞图标的初始化
            viewHolder.praise_button.setImageResource(R.mipmap.white);
            for(String user : commentInfo.getPraise_users()){
                if(user.equals(current_user)){
                    viewHolder.praise_button.setImageResource(R.mipmap.red);
                    break;
                }
            }
            //内部控件监听器
            viewHolder.praise_button.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    ImageButton imageButton = v.findViewById(R.id.item_praise_icon);
                    //点赞图标的变化
                    if(commentInfo.praise_count_change(current_user)) {
                        imageButton.setImageResource(R.mipmap.red);
                    }else{
                        imageButton.setImageResource(R.mipmap.white);
                    }
                    notifyDataSetChanged();
                    userSQLiteHelper.updateComment(commentInfo.getDate(), commentInfo.getPraise_users());
                }
            });
        }

        return convertView;
    }

    private static class ViewHolder
    {
        ImageView item_image;
        TextView item_username, item_date, item_comment, item_praise_count;
        ImageButton praise_button;
    }

}
```

### Click点击事件的添加

```java
commentAdapter = new CommentAdapter(this, username);

listView = findViewById(R.id.comment_list);
listView.setAdapter(commentAdapter);
listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
    @Override
    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
        //to do
    }
});
listView.setOnItemLongClickListener(new AdapterView.OnItemLongClickListener() {
    @Override
    public boolean onItemLongClick(AdapterView<?> parent, View view, final int position, long id) {
        if(commentAdapter.getItem(position).getUsername().equals(username)){
            //删除评论
                    new AlertDialog.Builder(CommentActivity.this).setTitle("Delete or not").setPositiveButton("YES",
                            new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialog, int which) {
                                    //Toast.makeText(getApplicationContext(),"对话框“确定”按钮被点击",Toast.LENGTH_SHORT).show();
                                    commentAdapter.removeItem(position);
                                }
                            }).setNegativeButton("NO",
                            new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialog, int which) {
                                    //Toast.makeText(getApplicationContext(),"对话框“取消”按钮被点击",Toast.LENGTH_SHORT).show();
                                }
                            }).create().show();
                }else{
            //举报评论
                    new AlertDialog.Builder(CommentActivity.this).setTitle("Report or not").setPositiveButton("YES",
                            new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialog, int which) {
                                    Toast.makeText(getApplicationContext(),"Report Successfully.",Toast.LENGTH_SHORT).show();
                                }
                            }).setNegativeButton("NO",
                            new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialog, int which) {
                                    //Toast.makeText(getApplicationContext(),"对话框“取消”按钮被点击",Toast.LENGTH_SHORT).show();
                                }
                            }).create().show();
                }
        //去除重复的单击事件
        return true;
    }
});
```

### 发送评论

```java
Button button = findViewById(R.id.comment_submit);
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        if(editText.getText().toString().isEmpty()){
            Toast.makeText(CommentActivity.this, "Comment is empty.",Toast.LENGTH_SHORT).show();
            return;
        }
        //获取系统日期
        commentAdapter.addItem(new CommentInfo(formatter.format(new Date(System.currentTimeMillis())), username, editText.getText().toString(),image,new ArrayList<String>()));
        //清空发送框中的内容
        editText.setText("");
    }
});
```

## 获取本机的通讯录

这里实现的是点击评论出现评论的**用户名**以及其**电话号码**的对话框

![4](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android7/4.gif)

首先需要**静态申请权限**，在`AndroidManifest.xml`里添加

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapplication">
    <uses-permission android:name="android.permission.READ_CONTACTS"/>
    <application
    ...
    </application>
</manifest>
```

但是还是需要**动态申请权限**

在`listview`的点击事件中添加

```java
//动态申请权限
requestPermissions(new String[]{Manifest.permission.READ_CONTACTS}, PERMISSIONS_REQUEST_READ_CONTACTS);
//得到手机号码
show_number = "\nPhone number not existed.";
Cursor cursor = getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,null, ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME + " = \"" + commentAdapter.getItem(position).getUsername() + "\"", null, null);
if(cursor!= null && cursor.moveToFirst()){
    show_number = "\nPhone: ";
    do {
        show_number += cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER)) + "         ";
    } while (cursor.moveToNext());
}
if(cursor != null)
    cursor.close();

new AlertDialog.Builder(CommentActivity.this).setTitle("Info").setMessage("Username: " + commentAdapter.getItem(position).getUsername() + show_number).setPositiveButton("OK",
        new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                //Toast.makeText(getApplicationContext(),"对话框“确定”按钮被点击",Toast.LENGTH_SHORT).show();
            }
        }).create().show();
```



但是出现了一些状况，就是运行到这一句的时候

```
Cursor cursor = getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,null, ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME + " = \"" + commentAdapter.getItem(position).getUsername() + "\"", null, null);
```

申请权限的对话框还没有弹出来，也就是说此时**权限还没有申请到**，但是却执行了搜索通讯录的操作，所以会出现以下报错

![1542125860646](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android7/1542125860646.png)



**解决办法：确保申请到了权限再显示电话号码**

修改单击事件

```java
listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
    @Override
    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
        //动态申请权限
        requestPermissions(new String[]{Manifest.permission.READ_CONTACTS}, PERMISSIONS_REQUEST_READ_CONTACTS);
        click_comment = commentAdapter.getItem(position);
    }
});
```

并添加申请结果的处理方法

```java
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions,
                                       int[] grantResults) {
    if (requestCode == PERMISSIONS_REQUEST_READ_CONTACTS) {
        if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            // Permission is granted
            //得到手机号码
            show_number = "\nPhone number not existed.";
            Cursor cursor = getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,null, ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME + " = \"" + click_comment.getUsername() + "\"", null, null);
            if(cursor!= null && cursor.moveToFirst()){
                show_number = "\nPhone: ";
                do {
                    show_number += cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER)) + "         ";
                } while (cursor.moveToNext());
            }
            if(cursor != null)
                cursor.close();

            new AlertDialog.Builder(CommentActivity.this).setTitle("Info").setMessage("Username: " + click_comment.getUsername() + show_number).setPositiveButton("OK",
                    new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            //Toast.makeText(getApplicationContext(),"对话框“确定”按钮被点击",Toast.LENGTH_SHORT).show();
                        }
                    }).create().show();
        } else {
            Toast.makeText(this, "Until you grant the permission, we canot display the names", Toast.LENGTH_SHORT).show();
        }
    }
}
```

这样子对话框在申请到了权限之后才会显示，就不会出错了！

## 笔记

添加图片到资源文件夹引用的时候又一次出现错误

```
java.lang.RuntimeException: Unable to start activity
ComponentInfo{com.example.myapplication/com.example.myapplication.MainActivity}:android.view.InflateException: Binary XML file line #47: Binary XML file line#47: Error inflating class ImageButton
```

**解决办法：**

把图片从`mipmap-anydpi-v26`放到`mipmap-hdpi`



## 项目结构

### 文件结构

![1542128122750](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android7/1542128122750.png)

### 数据结构

可以看到确实有一个数据库文件，有几个用户头像的副本



![1542127662720](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android7/1542127662720.png)



打开数据库查看



![1542127805022](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android7/1542127805022.png)

![1542127844289](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android7/1542127844289.png)

验证数据没有什么问题

## 完整代码

```java
package com.example.myapplication;

import android.content.ContentResolver;
import android.content.Intent;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageButton;
import android.widget.RadioButton;
import android.widget.RadioGroup;
import android.widget.Toast;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import android.net.Uri;
import java.util.UUID;

public class MainActivity extends AppCompatActivity {
    Boolean status = true;//false表示注册，true表示登录
    ImageButton register_image;
    String default_image = "android.resource://com.example.myapplication/" + R.mipmap.me;
    UserSQLiteHelper userSQLiteHelper;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //userSQLiteHelper.dropComment();
        userSQLiteHelper = new UserSQLiteHelper(this);
        final RadioGroup radioGroup = findViewById(R.id.radio_group);
        final EditText login_username = findViewById(R.id.login_username);
        final EditText login_password = findViewById(R.id.login_password);
        final EditText register_username = findViewById(R.id.register_username);
        final EditText register_password = findViewById(R.id.register_password);
        final EditText register_confirm_password = findViewById(R.id.register_confirm_password);
        register_image = findViewById(R.id.register_image);
        Button button_ok = findViewById(R.id.button_ok);
        Button button_clear = findViewById(R.id.button_clear);

        register_image.setImageURI(Uri.parse(default_image));

        radioGroup.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(RadioGroup group, int checkedId) {
                if(((RadioButton)findViewById(checkedId)).getText().toString().equals("Login")){
                    register_confirm_password.setVisibility(View.GONE);
                    register_image.setVisibility(View.GONE);
                    register_password.setVisibility(View.GONE);
                    register_username.setVisibility(View.GONE);

                    login_username.setVisibility(View.VISIBLE);
                    login_password.setVisibility(View.VISIBLE);
                    status = true;
                }else{
                    register_confirm_password.setVisibility(View.VISIBLE);
                    register_image.setVisibility(View.VISIBLE);
                    register_password.setVisibility(View.VISIBLE);
                    register_username.setVisibility(View.VISIBLE);

                    login_username.setVisibility(View.GONE);
                    login_password.setVisibility(View.GONE);
                    status = false;
                }
            }
        });

        register_image.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent();
                intent.setAction(Intent.ACTION_PICK);
                intent.setType("image/*");
                startActivityForResult(intent, 0);
            }
        });
        button_ok.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(!status){
                    if(register_username.getText().toString().isEmpty()){
                        Toast.makeText(MainActivity.this, "Username cannot be empty.",Toast.LENGTH_SHORT).show();
                        return;
                    }
                    if(register_password.getText().toString().isEmpty()){
                        Toast.makeText(MainActivity.this, "Password cannot be empty.",Toast.LENGTH_SHORT).show();
                        return;
                    }
                    if(!register_password.getText().toString().equals(register_confirm_password.getText().toString())){
                        Toast.makeText(MainActivity.this, "Password Mismatch.",Toast.LENGTH_SHORT).show();
                        return;
                    }
                    if(userSQLiteHelper.addUser(register_username.getText().toString(), register_password.getText().toString(), default_image)){
                        Toast.makeText(MainActivity.this, "Register Successfully.",Toast.LENGTH_SHORT).show();
                        return;
                    }else{
                        Toast.makeText(MainActivity.this, "Username already existed.",Toast.LENGTH_SHORT).show();
                        return;
                    }

                }else{
                    if(login_username.getText().toString().isEmpty()){
                        Toast.makeText(MainActivity.this, "Username cannot be empty.",Toast.LENGTH_SHORT).show();
                        return;
                    }
                    if(login_password.getText().toString().isEmpty()){
                        Toast.makeText(MainActivity.this, "Password cannot be empty.",Toast.LENGTH_SHORT).show();
                        return;
                    }
                    switch (userSQLiteHelper.loginVerify(login_username.getText().toString(), login_password.getText().toString())){
                        case 0:
                            Toast.makeText(MainActivity.this, "Login Successfully.",Toast.LENGTH_SHORT).show();
                            Intent intent = new Intent(MainActivity.this, CommentActivity.class);
                            intent.putExtra("username", login_username.getText().toString());
                            intent.putExtra("image",userSQLiteHelper.getImage(login_username.getText().toString()));
                            startActivity(intent);
                            break;
                        case 1:
                            Toast.makeText(MainActivity.this, "Username not existed.",Toast.LENGTH_SHORT).show();
                            break;
                        case 2:
                            Toast.makeText(MainActivity.this, "Invalid Password.",Toast.LENGTH_SHORT).show();
                            break;
                    }
                }
            }
        });
        button_clear.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(status){
                    login_username.setText("");
                    login_password.setText("");
                }else{
                    register_username.setText("");
                    register_password.setText("");
                    register_confirm_password.setText("");
                }
            }
        });

    }

    @Override

    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (data != null) {
            // 得到图片的全路径
            Uri uri = data.getData();
            ContentResolver cr = this.getContentResolver();
            //存储副本
            try {
                Bitmap bitmap = BitmapFactory.decodeStream(cr.openInputStream(uri));
                //通过UUID生成字符串文件名
                String image_name = UUID.randomUUID().toString() + ".jpg";
                //存储图片
                FileOutputStream out = openFileOutput(image_name, MODE_PRIVATE);
                bitmap.compress(Bitmap.CompressFormat.JPEG, 100, out);
                //获取复制后文件的uri
                Uri image_file_uri = Uri.fromFile(getFileStreamPath(image_name));
                //图片预览
                this.register_image.setImageURI(uri);
                //保存该URI
                default_image = image_file_uri.toString();
                out.flush();
                out.close();
            } catch (FileNotFoundException e) {
                Log.e("FileNotFoundException", e.getMessage(),e);
            } catch (IOException e) {
                Log.w("IOException", e.getMessage(), e);
            }
        }
        super.onActivityResult(requestCode, resultCode, data);
    }
}
```

```java
package com.example.myapplication;

import android.Manifest;
import android.app.Dialog;
import android.content.DialogInterface;
import android.content.pm.PackageManager;
import android.database.Cursor;
import android.provider.ContactsContract;
import android.support.v7.app.AlertDialog;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.AdapterView;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ListView;
import android.widget.Toast;

import org.w3c.dom.Text;

import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;

public class CommentActivity extends AppCompatActivity {
    ListView listView;
    String username;
    String image;
    String show_number = "\nPhone number not existed.";
    CommentInfo click_comment;
    CommentAdapter commentAdapter;
    SimpleDateFormat formatter = new SimpleDateFormat("yyyy年-MM月-dd日 HH:mm:ss");
    private static final int PERMISSIONS_REQUEST_READ_CONTACTS = 100;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_comment);

        username = getIntent().getStringExtra("username");
        image = getIntent().getStringExtra("image");

        commentAdapter = new CommentAdapter(this, username);

        listView = findViewById(R.id.comment_list);
        listView.setAdapter(commentAdapter);
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                //动态申请权限
                requestPermissions(new String[]{Manifest.permission.READ_CONTACTS}, PERMISSIONS_REQUEST_READ_CONTACTS);
                click_comment = commentAdapter.getItem(position);
            }
        });
        listView.setOnItemLongClickListener(new AdapterView.OnItemLongClickListener() {
            @Override
            public boolean onItemLongClick(AdapterView<?> parent, View view, final int position, long id) {
                if(commentAdapter.getItem(position).getUsername().equals(username)){
                    new AlertDialog.Builder(CommentActivity.this).setTitle("Delete or not").setPositiveButton("YES",
                            new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialog, int which) {
                                    //Toast.makeText(getApplicationContext(),"对话框“确定”按钮被点击",Toast.LENGTH_SHORT).show();
                                    commentAdapter.removeItem(position);
                                }
                            }).setNegativeButton("NO",
                            new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialog, int which) {
                                    //Toast.makeText(getApplicationContext(),"对话框“取消”按钮被点击",Toast.LENGTH_SHORT).show();
                                }
                            }).create().show();
                }else{
                    new AlertDialog.Builder(CommentActivity.this).setTitle("Report or not").setPositiveButton("YES",
                            new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialog, int which) {
                                    Toast.makeText(getApplicationContext(),"Report Successfully.",Toast.LENGTH_SHORT).show();
                                }
                            }).setNegativeButton("NO",
                            new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialog, int which) {
                                    //Toast.makeText(getApplicationContext(),"对话框“取消”按钮被点击",Toast.LENGTH_SHORT).show();
                                }
                            }).create().show();
                }
                //去除重复的单击事件
                return true;
            }
        });
        final EditText editText = findViewById(R.id.comment_edit_text);
        editText.setHint(username);
        Button button = findViewById(R.id.comment_submit);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(editText.getText().toString().isEmpty()){
                    Toast.makeText(CommentActivity.this, "Comment is empty.",Toast.LENGTH_SHORT).show();
                    return;
                }
                commentAdapter.addItem(new CommentInfo(formatter.format(new Date(System.currentTimeMillis())), username, editText.getText().toString(),image,new ArrayList<String>()));
                //清空发送框中的内容
                editText.setText("");
            }
        });
    }
    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions,
                                           int[] grantResults) {
        if (requestCode == PERMISSIONS_REQUEST_READ_CONTACTS) {
            if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // Permission is granted
                //得到手机号码
                show_number = "\nPhone number not existed.";
                Cursor cursor = getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,null, ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME + " = \"" + click_comment.getUsername() + "\"", null, null);
                if(cursor!= null && cursor.moveToFirst()){
                    show_number = "\nPhone: ";
                    do {
                        show_number += cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER)) + "         ";
                    } while (cursor.moveToNext());
                }
                if(cursor != null)
                    cursor.close();

                new AlertDialog.Builder(CommentActivity.this).setTitle("Info").setMessage("Username: " + click_comment.getUsername() + show_number).setPositiveButton("OK",
                        new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                //Toast.makeText(getApplicationContext(),"对话框“确定”按钮被点击",Toast.LENGTH_SHORT).show();
                            }
                        }).create().show();
            } else {
                Toast.makeText(this, "Until you grant the permission, we canot display the names", Toast.LENGTH_SHORT).show();
            }
        }
    }
}
```

