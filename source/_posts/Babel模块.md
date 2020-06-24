---
title: Babel模块
date: 2019-12-31 17:50:14
tags:
---

### @babel/core

### @babel/traverse

负责维护整棵树的状态，并且负责替换、移除和添加节点。

### babylon

Babel 的解析器

### @babel/types

一个用于 AST 节点的 Lodash 式工具库， 它包含了构造、验证以及变换 AST 节点的方法。 该工具库包含考虑周到的工具方法，对编写处理 AST 逻辑非常有用

### @babel/generator

Babel 的代码生成器，它读取 AST 并将其转换为代码和源码映射（sourcemaps）

### @babel/template

另一个虽然很小但却非常有用的模块。 它能让你编写字符串形式且带有占位符的代码来代替手动编码， 尤其是生成大规模 AST 的时候。 在计算机科学中，这种能力被称为准引用（quasiquotes）。
