---
title: Android手机应用开发（六） | 数据存储（上）
urlname: android6
tags:
  - Android
  - SharedPreference
copyright: true
category: Android
abbrlink: 5445
date: 2018-10-31 21:56:39
---

> 实验目的
>
> 1. 学习`SharedPreference`的基本使用。
> 2. 学习`Android`中常见的文件操作方法。
> 3. 复习`Android`界面编程。

------

## 实现效果

![GIF](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android6/GIF.gif)



## SharedPreference的使用

> 它是一个轻量级的存储类，特别适合用于保存软件配置参数。使用`SharedPreferences`保存数据，其背后是用xml文件存放数据，文件存放在`/data/data/<package name>/shared_prefs`目录下

### 先来创建一个简单的设置密码的页面

![1540994988442](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android6/1540994988442.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

   <EditText
       android:id="@+id/new_password"
       android:layout_marginTop="150dp"
       android:layout_width="match_parent"
       android:layout_height="60dp"
       android:hint="New Password"
       android:inputType="textPassword"
       app:layout_constraintTop_toTopOf="parent"
       />

    <EditText
        android:id="@+id/confirm_password"
        android:layout_marginTop="60dp"
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:hint="Confirm Password"
        android:inputType="textPassword"
        app:layout_constraintTop_toTopOf="@id/new_password"
        />
    <Button
        android:id="@+id/ok"
        android:layout_width="100dp"
        android:layout_height="50dp"
        android:text="OK"
        app:layout_constraintTop_toBottomOf="@id/confirm_password"/>

    <EditText
        android:id="@+id/login_password"
        android:layout_width="match_parent"
        android:visibility="invisible"
        android:hint="Password"
        android:inputType="textPassword"
        android:layout_height="60dp"
        app:layout_constraintTop_toBottomOf="@id/new_password"/>
    
    <Button
        android:id="@+id/clear"
        android:layout_width="100dp"
        android:layout_height="50dp"
        android:text="Clear"
        android:layout_marginStart="30dp"
        app:layout_constraintTop_toBottomOf="@id/confirm_password"
        app:layout_constraintStart_toEndOf="@id/ok"
        />

</android.support.constraint.ConstraintLayout>
```

其实这个页面是有三个`EditText`的，

**有两个是设置密码时的输入框（设置密码和确认密码）**

**有一个是登陆时的密码输入框（登录密码）**

此时还没有设置密码，所以另外一个`EditText`设置不可见`INVISIBLE`

### 处理按钮点击的事件

**文本清除按钮：**

```java
button_clear.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        //清空文本框中内容
        pw1.setText("");
        pw2.setText("");
        pw.setText("");
    }
});
```

**确定按钮：**

```java
button_ok.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        //还没注册密码成功
        if(pw.getVisibility() != View.VISIBLE){
            if(pw1.getText().toString().isEmpty()){
                Toast.makeText(MainActivity.this, "Password cannot be empty.", Toast.LENGTH_SHORT).show();
                //清空文本框中内容
                //模拟点击按钮
                button_clear.performClick();
            }else if(! pw1.getText().toString().equals(pw2.getText().toString())){
                Toast.makeText(MainActivity.this, "Password mismatch.", Toast.LENGTH_SHORT).show();
                button_clear.performClick();
            }else{
                 //简单两句代码，把密码写进存储中
                //getActivity()得到的就是MainActivity.this，不需要用这个函数了
                SharedPreferences sharedPref = MainActivity.this.getSharedPreferences("MY_PASSWORD", Context.MODE_PRIVATE);
                SharedPreferences.Editor editor = sharedPref.edit();
                //保存注册密码
                editor.putString("NEW_PASSWORD", pw1.getText().toString());
                //保存已注册状态
                editor.putString("STATUS", "LOGIN");
                //系统提醒我将commit改成apply的
                editor.apply();
                Toast.makeText(MainActivity.this, "Successfully.", Toast.LENGTH_SHORT).show();
                pw1.setVisibility(View.INVISIBLE);
                pw2.setVisibility(View.INVISIBLE);
                pw.setVisibility(View.VISIBLE);
            }
        }else{
           //获取密码比较是否正确
            SharedPreferences sharedPref = MainActivity.this.getSharedPreferences("MY_PASSWORD", Context.MODE_PRIVATE);
            String real_password = sharedPref.getString("NEW_PASSWORD", null);
            //if(!pw.getText().equals(real_password))我之前这样写是错误的
            if(!pw.getText().toString().equals(real_password)){
                //登录失败
                Toast.makeText(MainActivity.this, "Password incorrect", Toast.LENGTH_SHORT).show();
                button_clear.performClick();
            }else{
                //登陆成功
                Intent intent = new Intent(MainActivity.this, SecondActivity.class);
                startActivity(intent);
            }
        }

    }
});
```

输入密码之后，查看虚拟机的文件

`Logcat`->`Device File Explore`->`data/data/com.janking.project3.shared_prefs`->`MY_PASSWORD`

可以发现，确实存进去了密码，而且还存进去了一个关键字为`STATUS`的东西，这个是用来辨别是否已经设置了密码的

![1540996583921](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android6/1540996583921.png)

为了利用好这个`STATUS`字段，在`MainActivity`的`onCreate`方法里添加

```java
//这三句话也很重要，即使xml已经设置了三个可见度是，但是经过测试，中途删除了SharedPreference里面的文件的话输入框布局还是没变
pw1.setVisibility(View.VISIBLE);
pw2.setVisibility(View.VISIBLE);
pw.setVisibility(View.INVISIBLE);
//读取状态
SharedPreferences sp = MainActivity.this.getSharedPreferences("MY_PASSWORD", Context.MODE_PRIVATE);
if(sp != null){
    String last_status = sp.getString("STATUS", null);
    //不用判断是否是“LOGIN”，只要存在就行
    if(last_status != null){
        pw1.setVisibility(View.INVISIBLE);
        pw2.setVisibility(View.INVISIBLE);
        pw.setVisibility(View.VISIBLE);
    }
}
```



## Internal Storage文件的操作

在`MainActivity`设置密码之后，再登录成功之后，跳到另一个页面`SecondActivity`在这里简单使用一下文件的操作

### 布局

![1540997508834](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android6/1540997508834.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".SecondActivity">

    <EditText
        android:layout_margin="10dp"
        android:id="@+id/tv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:hint="Write like HelloWorld?"
        android:gravity="top"
        android:layout_weight="1"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:layout_gravity="fill"
        android:paddingStart="30dp"
        android:paddingEnd="30dp"
        android:orientation="horizontal">

        <!--按钮均匀分布-->
        <Button
            android:id="@+id/button_save"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:text="SAVE"
            android:layout_marginEnd="30dp"
            android:layout_height="match_parent" />

        <Button
            android:id="@+id/button_load"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:text="LOAD"
            android:layout_marginEnd="30dp"
            android:layout_height="match_parent" />

        <Button
            android:id="@+id/button_clear"
            android:layout_width="0dp"
            android:text="CLEAR"
            android:layout_weight="1"
            android:layout_height="match_parent" />

    </LinearLayout>

</LinearLayout>
```

### `LOAD`和`SAVE`按钮的事件处理

```java
//前面记得加
final String FILE_NAME = "data.txt";

//读取文件
bt_load.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        try (FileInputStream fileInputStream = openFileInput(FILE_NAME)) {
            byte[] contents = new byte[fileInputStream.available()];
            fileInputStream.read(contents);
            //et.setText(contents.toString())竟然不行
            et.setText(new String(contents));
            Toast.makeText(SecondActivity.this, "Load successfully.", Toast.LENGTH_SHORT).show();
        } catch (IOException ex) {
            Toast.makeText(SecondActivity.this, "Fail to load file.", Toast.LENGTH_SHORT).show();
        }
    }
});
//保存文件
        bt_save.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                try (FileOutputStream fileOutputStream = openFileOutput(FILE_NAME, MODE_PRIVATE)) {
                    fileOutputStream.write(et.getText().toString().getBytes());
                    Toast.makeText(SecondActivity.this, "Save successfully.", Toast.LENGTH_SHORT).show();
                } catch (IOException ex) {
                    Toast.makeText(SecondActivity.this, "Fail to save file.", Toast.LENGTH_SHORT).show();
                }
            }
        });
```

这其中有个小问题，就是读取的文件流是用`byte`数组接收的，开始我想把它转换为`String`用的是`toString`方法，但是得到的是乱码，然后改成`new String()`就可以了



> **toString()与new String ()用法区别**
>
> `str.toString`是调用了`object`对象的类的`toString`方法。一般是返回这么一个`String`：`[class name]@[hashCode]`。
> `new String(str)`是根据parameter是一个字节数组，使用`java`虚拟机默认的编码格式，将这个字节数组`decode`为对应的字符。若虚拟机默认的编码格式是`ISO-8859-1`，按照`ascii`编码表即可得到字节对应的字符。
>
> **什么时候用什么方法呢？**
>
> `new String（）`一般使用字符转码的时候,`byte[]`数组的时候
>
> `toString（）`将对象打印的时候使用 
>



经检验，文件内容确实写了进去（内容很乱，是我写的，而不是乱码……)，文件存放在`files`文件夹中

![1540997910417](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android6/1540997910417.png)



## 当 Activity 不可⻅时，如何将其从 activity stack 中除去（按返回键直接返回Home）

- 让不处于活动状态的`MainActivity`直接调用`finish`方法，点击返回按钮后会发现它就没了……

```java
//当此Acticity暂停时直接结束（移出栈）
@Override
protected void onPause() {
    super.onPause();
    MainActivity.this.finish();
}
```

- 还有一种办法，在`manifests`文件中添加

`android:noHistory="true"`

![1540998127063](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android6/1540998127063.png)

##  External Storage文件的操作

在`manifests`中添加

```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

![1541001942041](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android6/1541001942041.png)

进入`sdk`目录下的`tools`文件夹，输入下列命令

`.\mksdcard 128M sdcard.img`

即可创建一个`128M`大小的命名为`sdcard`的映像文件，它可以挂载到虚拟机上作为`sd`卡目录

![1541001233162](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android6/1541001233162.png)

添加一个图片文件到`mipmap`中，这里我加了我的头像`face.jpg`

在`SecondActivity`中新建一个方法

```java
void createExternalStoragePrivateFile(){
    File file = new File(getExternalFilesDir(null), "DemoFile.jpg");
    try {
        @SuppressLint("ResourceType") InputStream is = getResources().openRawResource(R.mipmap.face);
        OutputStream os = new FileOutputStream(file);
        byte[] data = new byte[is.available()];
        is.read(data);
        os.write(data);
        is.close();
        os.close();
    } catch (IOException e) {
        // Unable to create file, likely because external storage is
        // not currently mounted.
        Log.w("ExternalStorage", "Error writing " + file, e);
    }
}
```

并在`onCreate`中调用它

应用启动到`SecondActivity`中时，查看文件确实存在

![1541001890672](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/android6/1541001890672.png)

<!-- more --> 