---
title: mac 开发工具
date: 2017-12-21 10:09:17
tags:
---

### Java
现在 OS X 都不会自带 JDK 了，所以进行 Java 开发的话，需要下载 JDK。在 brew-cask 之前，我们需要从 https://developer.apple.com/downloads/ 或者 Oracle 网站上下载。还有更麻烦的－－卸载 JDK 和升级 JDK。

而 brew-cask 提供了自动安装和卸载功能，能够自动从官网上下载并安装 JDK 8。

``` bash
brew cask install java
```

如果需要安装 JDK 7 或者 JDK 6，可以使用`homebrew-cask-versions`：

``` bash
brew tap caskroom/versions
brew cask install java6
```

在 OS X 上，你可以同时安装多个版本的 JDK。你可以通过命令/usr/libexec/java_home -V来查看安装了哪几个 JDK。

那问题来了，当你运行java或者 Java 程序时使用的是哪个 JDK 呢？在 OS X 下，java也就是/usr/bin/java在默认情况下指向的是已经安装的最新版本。但是你可以设置环境变量JAVA_HOME来更改其指向：

`~/.bash_profile`
``` bash
export JAVA_6_HOME=`/usr/libexec/java_home -v 1.6`
export JAVA_7_HOME=`/usr/libexec/java_home -v 1.7`
export JAVA_8_HOME=`/usr/libexec/java_home -v 1.8`
export JAVA_HOME=$JAVA_7_HOME
#alias 命令动态切换JDK版本
alias jdk6=`export JAVA_HOME=$JAVA_6_HOME`
alias jdk7=`export JAVA_HOME=$JAVA_7_HOME`
export PATH=${JAVA_HOME}/bin:$PATH
```

其中JAVA_HOME=/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home可以用``JAVA_HOME=`/usr/libexec/java_home -v 1.6``这种更加通用的方式代替。
