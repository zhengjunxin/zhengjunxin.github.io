---
title: 高性能滚动（一）：Passive event listeners
date: 2017-01-15 16:26:43
categories: 专业技能
tags: 性能优化
---

简介：Passive event listeners 是新增的特性，指在 addEventListener 时传入{passive: true} 参数，表明不会在回调中执行 preventDefault()从而消除由 touch 和 wheel 事件带来的滚动阻塞，提高页面的滚动性能。

<!-- more-->
## 目录
* 问题原因
* 使用
* 特性检测

## 问题原因
所有的现代浏览器都有个线程专门用于滚动，保证页面滚动的流畅。但是因为 touchstart, touchmove 和 wheel （注意不包括 scroll 事件，因为 scroll 事件是在页面**已经**滚动时才触发的）这 3 个事件可以通过调用 preventDefault() 阻止滚动行为。所以如果为滚动元素注册了以上的事件，则浏览器**必须**等待回调函数执行完毕才能滚动，这无疑影响了滚动性能。而且即使是一个 [空的回调函数](http://rbyers.github.io/janky-touch-scroll.html)也会影响滚动性能。

根据Google统计
> For instance, in Chrome for Android 80% of the touch events that block scrolling never actually prevent it. 10% of these events add more than 100ms of delay to the start of scrolling, and a catastrophic delay of at least 500ms occurs in 1% of scrolls.  

所以 Passive event listeners 就是提供给我们一个方法，在用 addEventListener 注册 touchstart, touchmove 和 wheel 监听器时，承诺我们不会调用 preventDefault(), 让浏览器不需等待回调执行，马上开始滚动。

## 使用
```javasript
// passive touchstart event
target.addEventListener('touchstart', listener, {passive: true})
```
[MDN addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)也已经更新了该语法

## 特性检测
由于旧版本的浏览器 addEventListener 的第三个参数是接受一个布尔值的，如果传对象进去，只会被解析为 true，所以在注册时需要检测是否支持该特性，可以通过 [detect-passive-events](https://github.com/rafrex/detect-passive-events) 库检测
