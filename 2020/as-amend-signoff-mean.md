---
2020-02-22 17:49
---

# AndroidStudio/Idea 的 Amend commit和Sign-off commit 是什么意思？

![20200222173523816](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200222173523816.png)
如上图，直接做实验验证

## Sign-off commit
### 提交
创建一个新的测试类来进行提交，添加一个成员变量
![20200222173658543](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200222173658543.png)

### 结果
![20200222173804970](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200222173804970.png)
可以发现，就是在`commit`的信息后面加了一行签名，仅此而已

## Amend commit
### 提交
再次添加一个成员变量，进行提交
![20200222174042572](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200222174042572.png)

### 结果
![20200222174134758](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200222174134758.png)
似乎什么也没有发生，但是，打开修改记录会发现，该次的修改包含了上次的修改，上次的`commit`记录消失了， **本地commit替代了上次的提交记录**
![20200222174230910](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200222174230910.png)

## 总结
`Sign-off commit`和`Amend commit` 其实没什么关系

`Sign-off commit`就是在`commit message`后面加了一行签名

`Amend commit`是修改上一次的提交，可以用于某次提交不完整的时候，不需要再多一个修改的提交导致`git log`很复杂，只需要在修改的时候勾上`Amend commit`