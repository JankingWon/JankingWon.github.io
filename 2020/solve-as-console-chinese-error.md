## 背景
在`AndroidStudio`中新建了一个`Java Module`，但是点击 `Run ‘app’`之后，`Build Output` 控制台输出的中文都是乱码，都是问号一样的字符
![20200310125404524](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200310125404524.png)
`google`了很多方法，要么就是文不对题，要么就是各种抄，没有真正测试过！

### 错误方案一
 `File Encodings` 改为`UTF-8`？
 **没用！**
![20200310125650298](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200310125650298.png)

### 错误方案二
`build.gradle` 添加如下代码？
```
 tasks.withType(JavaCompile) {
    options.encoding = "UTF-8"
}
```

**没用！** 这是解决`System.out.print`输出的中文乱码问题的！



## 正确解决办法

 - 双击`Shift`，输入`vmoption`,，选择`Edit Custom CM Options`!![20200310131903457](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200310131903457.png)
 - 如果之前没有配置过，会弹出窗口问是否创建配置文件，点击`Create`
![20200310132047816](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200310132047816.png)
- 输入

```
-Dfile.encoding=UTF-8
```
![20200310132226615](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200310132226615.png)

 - 保存，重启就可以了！