---
title: flexible
date: 2017-09-26 17:56:01
tags:
---

### 自动计算函数
1.先设置编码格式，要不容易编译出现乱码。

2.使用时引用下面函数自动编译成rem单位；目前Flexible会将视觉稿分成100份（主要为了以后能更好的兼容vh和vw），而每一份被称为一个单位a。同时1rem单位被认定为10a。例如：当设计图为750px宽时，$base-font-size: 75px;当设计图为640px宽时，$base-font-size: 64px;padding-top: px2em(6px); 6px为设计图量出时大小。

``` less
@charset "UTF-8";

@function px2em($px, $base-font-size: 75px) {
  @if (unitless($px)) {
    @warn "Assuming #{$px} to be in pixels, attempting to convert it into pixels for you";
    @return px2em($px + 0px); // That may fail.
  } @else if (unit($px) == em) {
    @return $px;
  }
  @return ($px / $base-font-size) * 1rem;
}
```
