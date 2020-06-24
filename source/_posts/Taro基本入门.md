---
title: Taro基本入门
date: 2020-02-23 10:15:47
tags:
---

### Taro 版本对比

整个 1.x 版本 Taro 作为一个小程序转椅框架主要提供的功能有，支持微信小程序，支付宝小程序，字节跳动（头条）小程序，支持快应用和 QQ 小程序， H5， React Native，其它功能还提供了微信小程序转 Taro 代码，开放多端 UI 库打包能力，推出了首个可以跨多端使用的多端 UI 库 Taro UI。

其它：

- CSS Modules 的支持
- MobX 的支持
- 全面支持 JSX 语法和 HOOKS

参考资料
[Taro 1.3 震撼升级：全面支持 JSX 语法和 HOOKS](https://aotu.io/notes/2019/06/13/taro-1-3/)
[Taro 1.2：将已有微信小程序转换为多端应用](https://aotu.io/notes/2018/12/17/taro-1-2/)
[Taro 深度开发实践](https://aotu.io/notes/2018/11/30/taro_practice/)

2.x 版本 Taro 主要使用 Webpack 来实现编译构建，当中只会做区分编译平台、处理不同平台编译入参等操作，随后再调用对应平台的 runner 编译器 做代码编译操作，而原来大量的 AST 语法操作将会改造成 Webpack Plugin 以及 Loader，交给 Webpack 来处理。

参考资料
[Taro 2.0](https://aotu.io/notes/2020/01/08/taro-2-0/)

Taro Next 版本 将同时支持 React/Vue/Nerv 三种框架，不限制语言、语法

[Taro Next 发布预览版](https://aotu.io/notes/2020/02/03/taro-next-alpha/)

### Taro 基本使用

[Doc](https://taro-docs.jd.com/taro/docs/README.html)
