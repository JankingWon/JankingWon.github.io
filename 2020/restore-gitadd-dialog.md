## 背景
AS默认新建文件后会弹出一个「Add File to Git」的对话框，想让它不再提示，于是就点了一个「Remenber, dont't ask again」，问题是后来手抖点到了「Cancel」，以后添加文件都不会加到暂存区了，需要手动Add，那该 怎么办呢？
![20200528150234575 (1)](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200528150234575%20(1).png)

## 解决
* 办法一：使用快捷键**Ctrl + Alt + A**手动快速Add File，不过该快捷键跟QQ的截图快捷键有冲突
* 办法二：修改设置
Version Control -> Confimation中有个「When files are created」的设置
	* 「Show options before adding to version control」弹窗询问
	*  「Add silently」每次都添加
	* 「Do not add」每次都不添加
![20200528150759136](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200528150759136.png)