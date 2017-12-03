---
title: macrotask 和 microtask
date: 2017-12-03 21:24:21
tags:
---

### 简介

一个事件循环(EventLoop)中会有一个正在执行的任务(Task)，而这个任务就是从 macrotask 队列中来的。在whatwg规范中有 queue 就是任务队列。当这个 macrotask 执行结束后所有可用的 microtask 将会在同一个事件循环中执行，当这些 microtask 执行结束后还能继续添加 microtask 一直到真个 microtask 队列执行结束。
