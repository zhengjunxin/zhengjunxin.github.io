---
title: event-loop
date: 2017-03-08 19:09:11
tags:
- 前端
- 引擎相关知识
---

<!-- more -->
简介：学习事件循环的 macrotask 和 microtask；学习抛出异常后 JS 引擎的工作

## 事件循环
```javascript
setTimeout(function(){console.log(4)},0);
new Promise(function(resolve){
    console.log(1)
    for( var i=0 ; i<10000 ; i++ ){
        i==9999 && resolve()
    }
    console.log(2)
}).then(function(){
    console.log(5)
});
console.log(3);

// 1 2 3 5 4
```

浏览器只有**一个**事件循环（Event Loop），但可以有多个队列，每个队列的优先级不一样。浏览器先执行一个 macrotask，然后执行**所有** microtask
* macrotask 队列：整个脚本，setTimeout
* microtask 队列：then

1. 整个脚本在 macrotask
2. setTimeout 推入 macrotask
3. console.log(1)
4. resolve 导致 then 推入 microtask
5. console.log(2)
6. console.log(3)
7. 脚本执行完成毕，执行 microtask
8. console.log(5)
9. 执行下一个 macrotask
10. console.log(4)


## 异常
```javascript
setTimeout(() => {
    console.log(23333)
}, 0)

throw "Error"

// 结果 
// Uncaught Error
// 23333
// 而不是 Uncaught Error 后就不响应了
```
异常只是从当前函数中退出，但是引擎会**继续**工作（以前还以为抛出异常后，引擎就不响应了呢）
