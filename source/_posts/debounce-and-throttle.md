---
title: debounce 和 throttle 总结
date: 2017-03-17 19:48:57
tags: 
    - 前端
---
简介：介绍 debounce 和 throttle 的区别和使用场景
<!-- more -->
## 为什么需要节流函数
某些操作(如：输入监听、滚动监听)，会在短时间内频繁调用函数，造成页面卡顿。所以这时就需要节流函数来减少函数的调用频率，优化体验。但是节流函数可以分为防抖动(debounce)，节流(throttle)两种，那什么时候用防抖动，什么时候用节流呢？

## debounce 函数
debounce 把多个连续的函数调用汇总起来，然后在最后一次调用后触发。

{% img /images/debounce.jpg %}
  

联想：debounce 防抖动，尿完尿之后，才抖一下，而且只抖一下
使用场景：
* 输入监听，最后一次输入才是有效的输入

## throttle 函数
throttle 限制函数调用，在固定时间内只会触发一次

{% img /images/throttle.jpg %}
  

联想：throttle 节流器，掐紧水管，固定的时间只能滴一滴
使用场景：
* 滚动监听，此时不能用 debounce 因为他会在停下来时才触发

## 总结
debounce 是在多次函数调用，但只有一次是有效时使用。throttle 是在多次函数调用且都有效，但需要减少频率时使用。或者说只需要抖一抖时用 debounce 函数，其余的就用 throttle 函数。

## 参考
[debouncing-throttling-explained-examples](https://css-tricks.com/debouncing-throttling-explained-examples/
)
[函数节流和函数去抖应用场景辨析](https://github.com/hanzichi/underscore-analysis/issues/20)
