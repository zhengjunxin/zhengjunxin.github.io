---
title: 属性访问检测
date: 2017-01-08 10:31:12
categories: 专业技能
tags:
---

简介：发现定义 Object.defineProperty 的 getter 方法可以做很有意思的事，适合做浏览器的兼容性检测，特此记录一下
<!-- more -->

事情是这样的，为了优化浏览器的滚动体验， addEventListener 的第三个参数可以传一个包含 passive 属性的对象，表明该事件是 passive 的（具体可以看[Passive Event Listener](https://github.com/WICG/EventListenerOptions/blob/gh-pages/explainer.md)）。但是为了兼容旧版本的浏览器，就需要探测浏览器浏览器是否接受一个包含 passive 属性的对象。那么怎么探测呢？具体代码如下
```javascript
// Test via a getter in the options object to see if the passive property is accessed
var supportsPassive = false;
try {
  var opts = Object.defineProperty({}, 'passive', {
    get: function() {
      supportsPassive = true;
    }
  });
  window.addEventListener("test", null, opts);
} catch (e) {}

// Use our detect's results. passive applied if supported, capture will be false either way.
elem.addEventListener('touchstart', fn, supportsPassive ? { passive: true } : false);
```

自定义对象的属性和 getter 方法，如果被检测的函数使用了该属性，那么必定会调用 getter 方法，这样就能判断浏览器是是支持该特性的，真是巧妙到没有朋友啊！

## 总结
* 使用场景：检测是否访问过对象的某属性
* 使用方法：通过 Object.defineProperty 自定义 getter 方法来检测
* 原理：如果访问该属性，则会调用 getter 方法

## 自测题目
* 检测函数 say 是否支持接受 name 属性和 test 属性
```javascript
function say(person) {
    console.log(`${person.name} is ${person.age} old`)
}
```

### 答案
```javascript
function say(person) {
    console.log(`${person.name} is ${person.age} old`)
}

let isSupportName = false
try {
    const snooper = Object.defineProperty({}, 'name', {
        get() {
            isSupportName = true
        }
    })
    say(snooper)
}
catch(e) {}

let isSupportTest = false
try {
    const snooper = Object.defineProperty({}, 'test', {
        get() {
            isSupportTest = true
        }
    })
    say(snooper)
}
catch(e) {}
```

