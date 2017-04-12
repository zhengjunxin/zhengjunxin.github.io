---
title: debounce 和 throttle 总结
date: 2017-03-17 19:48:57
tags: 
    - 前端
---
简介：总结 debounce 和 throttle 函数的使用场景
<!-- more -->
## debounce
在最后一次调用后触发。
```javascript
// 1 表示debounced-function 的调用，0表示空闲，X表示真正的执行
// 在 delay = 2s 时，在最后一次调用后的 2s 后执行
111100
{% img /images/debounce_edited.jpg %}
      x
```

联想：debounce 防抖动，尿完尿后，才抖一下

使用场景：
* 监听搜索关键字改变（只在最后一次输入后才是有效的）


## throttle
每隔一定时间触发。
```javascript
// 1 表示throttled-function 的调用，0表示空闲，X表示真正的执行
// 在 delay = 2 时，每隔 2s 调用一次
111111
  x  x
```
{% img /images/throttle_edited.jpg %}

联想：throttle 节流器，掐紧水管，每 delay 秒只滴一滴

使用场景：
* 滚动检查（此时不能用 debounce 因为他会在停下来时才触发）
