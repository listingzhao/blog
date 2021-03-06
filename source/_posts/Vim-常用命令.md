---
title: Vim 常用命令
date: 2017-09-18 10:47:52
tags:
---


### 安装vim
``` bash
$ apt-get update
$ apt-get install vim  # ubuntu
$ yum upgrade
$ yum install vim      # centos
```

### 显示行号
``` bash
$ :set nu
```

### 多行注释
1. 进入命令行模式，按ctrl + v进入 visual block模式（可视快模式），然后按j, 或者k选中多行，把需要注释的行标记起来
2. 按大写字母I，再插入注释符，例如//
3. 按esc键就会全部注释了（我的是按两下）

``` bash
$ :10,20s/^/#/g  #查找替换 表示将10-20行添加注释
```

### 取消多行注释
1. 进入命令行模式，按ctrl + v进入 visual block模式（可视快模式），按小写字母l横向选中列的个数，例如 // 需要选中2列
2. 按字母j，或者k选中注释符号
3. 按d键就可全部取消注释

``` bash
$ :10,20s/^#//g  #查找替换 表示将10-20行取消注释
```

### 多行缩进
按v进入visual状态，选择多行，用>或<缩进或缩出

### 删除多行
``` bash
$ :32,65d    # 删除多行
```
