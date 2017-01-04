---
title: 手Q演出选座页性能10倍提升的优化
date: 2017-01-02 21:27:38
categories: 专业技能
tags:
  - React
---

简介：本文将会分析在使用React时，面对多组件（如1K个组件）的初始化渲染，和单组件更新的场景，可以使用的优化思路

<!-- more -->
## 目录
* 场景
* single connect方案
* multiple connect方案
* smart seat方案
* canvas seat方案
* 总结

## 场景
在我们演出项目的选座页，需要展示演出的座位，并且在用户选择某一座位后，要更新该座位的视图。
为了简化问题，在这里抽象一下模型，即：有Seats和Seat两个组件
* Seats从store中接收一个包含1000个包含座位信息的seat元素的数组seats，然后渲染Seat组件
* Seat组件被点击后，变更该组件颜色

## Single Connect方案
[Single Connect方案](https://joezheng2015.github.io/learningReact/#/canvasproject/singleconnect) ，这是我们最开始的方案，只完成了基本优化，即在Seat组件内定义shouldComponentUpdate减少不必要的render。每次点击后的组件更新

### 性能
* 初始渲染时间：iphone7平均40ms，小米平均474ms
* 组件更新时间：iphone7平均70ms，小米平均123ms

### 分析
此时更新时的数据流是：Seat被点击dispatch一个action更新store。Seats组件在接受新的seats数组后，调用render，然后map更新Seat组件

## Multiple Connect方案
[Multiple Connect方案](https://joezheng2015.github.io/learningReact/#/canvasproject/multipleconnect)是参考Dan神提出的 [分离数据](https://github.com/mweststrate/redux-todomvc/pull/1) 思想，来减少不必要的render。
实现思想是：在每次更新时，其实Seats是不需要render的，真正需要render的只有那个被点击的Seat。
实现方式是：
* Seats组件只关心要渲染多少个Seat组件，即从seats数组中分离出seatIds表示每个元素的index，然后Seats组件根据seatIds来map，注意因为每次更新seatIds是不会变的，所以可以 _减少Seats的渲染_
* Seat组件只关心自己的状态，Seat组件也connect store，并且根据Seats传进来的id来从seats数组中获得自己对应的seat数据

### 性能
* 初始渲染时间：iphone7平均103ms，小米平均1700ms
* 组件更新时间：iphone7平均8ms（8x提升），小米平均17ms（7x提升）
* 总结：初始化性能降低，但组件更新性能提升了7x左右

### 分析
* 更新时的数据流变成：Seat被点击dispatch一个action更新store。Seats组件关心的seatIds不变，所以不用更新。对应的Seat组件订阅到自己的seat发生变化，才需要调用render。所以整个流程只有一个Seat组件调用了render，加快了更新时间。但是因为Seat组件也connect store，在初始化带来额外的开销

* 虽然提升了组件的更新性能，但降低了初始化性能，有点得不偿失

## Smart Seat方案
[Smart Seat方案](https://joezheng2015.github.io/learningReact/#/canvasproject/smartseat)的主要思想是：把Seat由单纯的展示型组件，变成功能型组件，由每个Seat来管理自己的状态。
实现方式如下：在Seat组件被点击时，dispatch一个action。 _然后在回调里setState_触发组件更新
```javascript
onClick = id => {
    this.props.selectSeat(id)
        .then(selectSuccess => {
            this.setState({
                selected: selectSuccess,
            })
        })
}
```

### 性能
* 初始渲染时间：iphone7平均41ms，小米平均478ms
* 组件更新时间：iphone7平均9ms（8x提升），小米平均5ms（24x提升）
* 总结：初始化性能不变，但组件更新性能提升了16x左右

### 分析
由于初始渲染逻辑没变，初始化性能没有下降。但组件的更新由回调驱动，而不用经过store, Seats的层层传递，而且只有对应Seat组件会接收到回调，所以更新性能提升了

## Canvas Seat方案
[Canvas Seat方案](https://joezheng2015.github.io/learningReact/#/canvasproject/canvasseat)的思想是使用canvas来绘制座位和更新对应座位，这样就可以不用渲染那么多Seat节点，只渲染一个canvas节点即可。规避dom慢的缺陷。
实现方式：在Seats组件只render一个canvas节点，并在componentDidMount后绘制座位

### 性能
* 初始渲染时间：iphone7平均4.3ms（9x提升），小米平均42ms（11x提升）
* 组件更新时间：iphone7平均3.7ms（18x提升），小米平均10ms（12x提升）
* 总结：初始化性能提升10x左右，组件更新性能提升了15x左右

### 分析
由于初始时减少大量节点的渲染，初始化性能惊人的提升了10x！而且更新组件时，也只是重新绘制一个座位，组件更新性能也得到了极大的18x的提升

## 总结
在摸索React性能优化的过程中，越发的理解前端的性能瓶颈不在JS，而在dom的操作。我曾尝试过各种方法减少数组的遍历，更高效的取整（~~替代parseInt），但效果都是微乎其微。而每一次减少组件的render，却能给性能带来成倍的提升，如Multiple Connect和Smart Seat方案中，减少Seats组件的渲染带来了5x的提升。Canvas Seat方案中，减少了999个组件的渲染（1个canvas代替1000个Seat），带来10x的性能提升！

而且，另一个让我感受很深的就是，除了Multiple Connect方案是根据Dan神提出的思想实现的，其余2个方案都是我们手Q队员提出的，而且性能都比Dan神的方案好！这让我学习到，作为一个程序员，能因地制宜，发挥自己的 _想象力_ 提出更针对性的方案更重要。
