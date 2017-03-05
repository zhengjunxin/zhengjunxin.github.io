---
title: Udacity 前端学习总结
date: 2017-02-13 18:01:17
tags: 前端
---
简介：Udacity 前端学习总结
<!-- more -->

## P1 HTML
* 学习了 `<em>` 与 `<i>` 的区别：视觉上都是斜体，但 `<em>` 表示强调， `<i>` 是从斜体，从文档中区分出来
* `<strong>` 和 `<b>` 的区别：视觉上都是粗体，但 `<strong>` 表示重要，`<b>` 则是以粗体从文档中区分出来
* `<mark>` 表示搜索结果高亮
* 用 `<del>` 表示删除的语义， `<strike>` 也可以但 HTML 5 不推荐
* `<sup>` 和 `<sub>` 表示上下标
* 总结：
``` html
逃避可耻<em>但</em>有用

<i>我就是斜体表示一下</i>

删除很<strong>危险</strong>

<b>我就是粗体表示一下</b>

搜索结果<mark>高亮</mark>了
```

## P6 网站优化
### 关键渲染路径
#### 将 HTML 转化为 DOM
主要流程是 Characters -> Tokens -> Nodes -> Dom

即解析 HTML 的字符，遇到 <> 标签就把它转化为一个令牌（注意令牌有起始、结束令牌之分，以此浏览器可以确定层级关系），另一个进程同时会消耗令牌（同步可以加快渲染速度吧）生成节点对象。当所有令牌消耗完，则 DOM 树也生成完毕

DOM 树表示了 HTML 的内容和属性，以及各节点间的关系。此时每个节点对象包含了 html 上定义的属性，如 img 节点对象包含了 html 上 img 的 src 属性
* 构建 DOM 是逐步的

#### 将 CSS 转化为 CSSOM
* 在 Timeline 的 Event Log 的 Rendering 的 Recalculate Style 可以看见应用 style 所用的时间

#### 渲染树
* 在用 CSSOM 和 DOM 构建渲染树时，是把可见的节点拷贝进渲染树。如 <head> 内的元素是不可见的，style 设为 display none 的元素是不可见的，都不会再渲染树中
* 有些元素虽然不占空间，但是也会存在渲染树中，如 height 为0 的元素，element: before 没有设置 content 的元素， visible 为 hidden 的元素

#### 布局
如果没有在 <meta> 中指定 viewport 大小，浏览器将使用默认的 viewport 大小，如苹果的 980 px

#### 优化关键路径
CSS 会阻塞**渲染**，因为浏览器需要构建完 render 树才会渲染，但不阻塞解析 DOM。但可以通过定义媒体查询 CSS 文件来减少阻塞。 [阻塞渲染的 CSS (by google)](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-blocking-css)

#### 有关 JavaScript 依赖的详细信息
JS 会阻塞**解析**。
CSS 阻塞 JS 执行，因为 JS 可能会更改 CSSOM，所以浏览器”智能“的等到 CSS 文件下载并构建成 CSSOM 后，才会执行 JS

#### 预加载扫描程序
[浏览器如何预加载](https://andydavies.me/blog/2013/10/22/how-the-browser-pre-loader-makes-pages-load-faster/)

#### RAIL
* Response：100ms, 反馈用户的交互
* Animate：60fps, 展示动画，只有这时才需要!
* Idle：50ms, 利用空闲时间，加载不重要的内容
* Load：1000ms, 首屏显示

#### 更改样式
[像素管道](https://developers.google.com/web/fundamentals/performance/rendering/)即更改样式时，要经历的 5 个区域
1. JavaScript（更改样式）
    * 使用 requestAnimationFrame
    * 使用 web worker
2. Style Calculation（样式计算），可分为两个步骤：匹配选择器、根据选择器计算样式
    * 降低选择器复杂度 BEM
    * 减少要计算样式的元素数目
3. Layout（重排）
    * 使用 flex 布局（比 flow 和 position 高效？！）
4. Paint（重绘）
    * 使用复合层，减少重绘范围
    * 降低样式复杂度如 box-shadow
5. Composite（复合）
    * 使用 transform 和 opacity 来做动画
    * 减少复合层
`transform: translate(0, 0)` 是个 2D translate 不会产生复合层，要 `transform: translateZ(0)` 表示 3D translate 才会产生复合层


#### 强制同步布局事件
该事件的发生是因为，样式发生了改变，浏览器不用同步更新，而是缓存起来批量更新样式。但是如果改变了样式，然后查询样式，则浏览器需要同步布局，才能返回正确的样式
```javascript
// 此时浏览器会强制同步布局，然后才能返回正确的 offsetHeight
box.classList.add('new-style')
box.offsetHeight

// 正确的做法的，如果不需要查询更新后的样式，可以把查询放在前面
box.offsetHeight
box.classList.add('new-style')
```