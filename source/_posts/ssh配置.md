---
title: ssh配置
date: 2019-06-09 09:49:52
tags:
---

### 简介

Mac os 下，ssh证书是默认存放在 ~/.ssh 目录下的，Linux系统也是如此。默认的ssh-key文件是：id_rsa 和 id_rsa.pub。

你可以使用一个ssh-key用于多个服务器之间的登录，也可以针对不同的服务器，使用不同的ssh-key文件。


### 生成证书

``` bash
ssh-keygen
```

### 将公钥安装到远程服务器上

``` bash
ssh-copy-id -i ~/.ssh/test_rsa.pub username@remote
```

### 修改config配置文件（自定义key需要）

``` bash
# 登录服务器：192.169.1.1，使用私钥test_rsa
Host 192.169.1.1
User username
IdentityFile ~/.ssh/test_rsa
```

### 登录
``` bash
ssh root@39.100.127.79
```
