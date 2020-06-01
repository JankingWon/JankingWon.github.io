## 总共分为两步
### 修改apk
使用`bandizip` 等软件直接打开`apk`，进行需要的修改，然后把**META-INF**文件夹中的**xxx.RSA**、**xxx.SF**和**xxx.MF**都删掉

> 或者解压apk到一个文件夹中进行修改并按照上述操作删除文件后，然后压缩成**zip**格式，压缩完成后把压缩文件后缀改为**apk**


### 重新签名apk
#### 方法一：拥有.keystore签名或.jks签名（生成V1签名）
执行命令（需要配置好`java`环境，`windows`也可以直接执行）

```
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore {签名文件} {apk文件名} {签名别名}
```
比如

```
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore janking.jks sample-release.apk janking
```
![20200220190148712](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200220190148712.png)
> `[签名文件]` 可以是`keystore`文件，也可以是`jks`文件

#### 如果没有java环境怎么办？
想办法安装咯
#### 顺便说一下怎么生成签名？
1. 点击`Android Studio` 菜单栏 `Build` 
2.  `Generate Signed Bundle or APK` 
3. 随便 选择APK 还是 Android App Bundle
4. `Create New ...`
5. `key store path` 选择一个密钥存储的文件，在弹出来的框中输入要生成密钥的文件名
6. ![20200220191900968](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200220191900968.png)
7. 完成！

#### 方法二、拥有.pem和.pk8签名（生成V2签名）
使用`SignApk.jar`（[下载地址](https://github.com/JankingWon/SignApk/releases)），输入命令

```
java -jar SignApk.jar [-w] {.pem文件} {.pk8文件} {未签名apk} {生成的apk}
```

> [-w] 参数表示对整个jar包签名

例如
```
java -jar SignApk.jar  platform.x509.pem platform.pk8　old.apk new.apk
```

#### 注意
如果之前安装过其他签名的该apk文件，修改签名后会安装失败，需要卸载之前已安装的应用