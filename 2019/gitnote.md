---
title: Git学习笔记
urlname: gitnote
tags:
  - Git
copyright: true
category: Git
date: 2019-02-07 20:45:59
---

## 常用命令

```bash
$ git pull github master --allow-unrelated-histories
允许没有提交历史的仓库合并
```

```bash
$ git fetch --all && git reset --hard origin/master && git pull
拉取远程仓库，强制覆盖
```

<!-- more --> 

## 参考资料

