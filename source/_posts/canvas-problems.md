---
title: canvas 绘制问题总结
date: 2017-05-26 10:48:50
tags:
  - canvas
---

简介：本文将会介绍在使用 canvas 过程中遇到的问题

<!-- more -->

## 遇到的问题
- 模糊问题
- IOS上 canvas 尺寸限制问题

## 模糊问题
模糊问题主要是 canvas 绘制到高清屏上时，被拉伸导致的。

举个简单的例子就是：一个 200*200 的 canvas，渲染在设备像素比为 1 的设备上，正常渲染，所以正常显示。但是在设备像素比为 2 的设备上，则需要 400*400 像素才能正常显示，现在只有 200*200 所以 canvas 被拉伸导致模糊了。

解决办法就是绘制 devicePixelRatio 倍的 canvas。比如设备像素比为 2，只要绘制 400*400 大小的 canvas，再定义样式缩小为 200*200。这样最终绘制时，即使被拉伸为 400*400，但 canvas 本身就是 400*400 所以就能正常显示啦~

代码如下：
``` javascript
const width = 200
const height = 200
const ratio = window.devicePixelRatio || 1

canvas.width = width * ratio
canvas.height = height * ratio
ctx.scale(ratio, ratio)
canvas.style.width = `${width}px`
canvas.style.height = `${height}px`
```

## IOS上 canvas 尺寸限制问题

描述问题
- IOS 上对 canvas 的尺寸有限制，超过限定尺寸后调用绘制方法不会绘制到 canvas 上，但不报错！
- 尺寸是指 canvas 的宽高，而不是样式宽高。即 canvas.width 而不是 canvas.style.width
- 限制是针对面积的，而且随 IOS 版本变化的，IOS9 以下最大允许 2096 * 2096 的 canvas，IOS9以上最大允许 4096 * 4096

### 解决思路1
绘制失败属于少见的异常场景，所以可以先绘制，再探测是否绘制成功，如果失败，则降低清晰度再绘制。这样能保证在大部分的正常场景能较快的完成绘制，小部分的异常场景绘制慢一点而已。

经过搜索，有 2 种方式可以探测是否绘制成功
- toDataURL
- getImageData

结果：在小米Note 上探测是否绘制成功会闪退。详细情况是：小米支持 16000*16000 尺寸的 canvas，但是在 7000*7000 尺寸以上调用 toDataURL 就会闪退，在 8500*8500 尺寸以上调用 getImageData 就会闪退。所以该方案失败！

### 解决思路2
发现安卓对 canvas 的限制比较松，如小米Note能支持到 16000*16000 的尺寸。所以可以检测设备机型，如果是安卓则正常绘制，如果是 IOS 则根据版本进行 canvas 缩放。保证不超过该版本允许的尺寸。

结果：在 IOS7 上绘制的尺寸临近但不超过 2096*2096 能成功绘制，但是会出现 canvas 中间莫名空白一块的奇异问题。所以该方案最终失败。

### 结论
IOS 上绘制大屏的 canvas 暂时只能通过绘制多个 canvas 实现

## 参考
[High DPI Canvas](https://www.html5rocks.com/en/tutorials/canvas/hidpi/)
[Maximum size image on Safari page]()
