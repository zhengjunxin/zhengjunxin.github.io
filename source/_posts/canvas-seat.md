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
* 前提
* 场景
* Single Connect 方案
* Multiple Connect 方案
* Smart Seat 方案
* Canvas Seat 方案
* 总结

## 前提
* 本文测试时所使用库的版本如下
```json
  "dependencies": {
    "classnames": "^2.2.5",
    "react": "^15.4.0",
    "react-dom": "^15.4.0",
    "react-redux": "^4.4.6",
    "react-router": "^3.0.0",
    "redux": "^3.6.0",
    "redux-logger": "^2.7.4",
    "redux-thunk": "^2.1.0",
    "reselect": "^2.5.4"
  }
```

## 场景
在我们演出项目的选座页，需要展示演出的座位，并且在用户选择某一座位后，要更新该座位的视图。为了简化问题，在这里抽象一下模型，即有 Seats 和 Seat 两个组件
* Seats 从 store 中接收一个包含 1000 个包含座位信息的 seat 元素的数组 seats ，然后渲染 Seat 组件
* Seat 组件被点击后，变更该组件颜色

## Single Connect 方案
[Single Connect方案](https://joezheng2015.github.io/learningReact/#/canvasproject/singleconnect) ，这是我们最开始的方案，只在 Seat 组件内定义 shouldComponentUpdate 减少不必要的 render

### 性能及分析
|  | iphone7 | 小米 |
| - | - | - |
| 初始渲染时间 | 40ms | 474ms |
| 组件更新时间 | 70ms | 123ms |

此时性能瓶颈主要有 2 点：每次更新时 Seats 组件都需要渲染；数据需要从 store 经过 Seats 最终才能传递给对应的 Seat 组件。后面优化也是针对这 2 点进行的

## Multiple Connect 方案
[Multiple Connect方案](https://joezheng2015.github.io/learningReact/#/canvasproject/multipleconnect)是参考 Dan 神提出的 [分离数据](https://github.com/mweststrate/redux-todomvc/pull/1) 思想，来减少不必要的 render

实现思想：
在每次更新时，其实 Seats 是不需要 render 的，真正需要 render 的只有那个被点击的 Seat

实现方式：
* Seats 组件只关心要渲染多少个 Seat 组件，即从 seats 数组中分离出 seatIds 表示每个元素的 index，然后 Seats 组件根据 seatIds 来 map，注意因为每次更新 seatIds 是不会变的，所以可以 _减少 Seats 的渲染_
* Seat 组件只关心自己的状态，Seat 组件也 connect store，并且根据 Seats 传进来的 id 来从 seats 数组中获得自己对应的 seat 数据

### 性能及分析
|  | iphone7 | 小米 |
| - | - | - |
| 初始渲染时间 | 103ms | 1700ms |
| 组件更新时间 | 8ms (8x) | 17ms (7x) |

组件更新性能平均提升了7x左右，因为此时减少了 Seats 组件的渲染，且数据直接从 store 通过 connect 传递给对应的 Seat 组件。但是 Seat 组件需要在初始化时 subscribe store，所以增加了初始渲染时间

## Smart Seat 方案
[Smart Seat方案](https://joezheng2015.github.io/learningReact/#/canvasproject/smartseat)的主要思想是：把 Seat 由单纯的展示型组件，变成功能型组件，由每个 Seat 来管理自己的状态

实现方式如下：在 Seat 组件被点击时，dispatch action 然后在回调里 setState 触发组件更新
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

### 性能及分析
|  | iphone7 | 小米 |
| - | - | - |
| 初始渲染时间 | 41ms | 478ms |
| 组件更新时间 | 9ms (8x) | 5ms (24x) |

此时不但减少 Seats 组件的渲染，且数据也绕过 store 直接从 actions 的回调中获得，使得组件更新性能平均提升了 16x 左右

## Canvas Seat 方案
[Canvas Seat方案](https://joezheng2015.github.io/learningReact/#/canvasproject/canvasseat)的思想是使用 canvas 来绘制座位和更新对应座位，这样就可以不用渲染那么多 Seat 节点，只渲染一个 canvas 节点即可，规避 dom 慢的缺陷。且更新不发生 render，只是重绘对应位置的座位，省去 Seats 组件的渲染

实现方式：在 Seats 组件只 render 一个 canvas 节点，并在 componentDidMount 后绘制座位

### 性能及分析
|  | iphone7 | 小米 |
| - | - | - |
| 初始渲染时间 | 4.3ms (9x) | 42ms (12x) |
| 组件更新时间 | 3.7ms (18x) | 10ms (12x) |

初始时减少大量节点的渲染，初始化性能惊人的提升了 10x。更新组件时，也只是重新绘制一个座位，组件更新性能也得到了极大的 18x的提升

## 总结
在摸索 React 性能优化的过程中，越发的理解前端的性能瓶颈不在 JS，而在 dom 的操作。我曾尝试过各种方法减少数组的遍历，更高效的取整（ ~~ 替代 parseInt ），但效果都是微乎其微。而每一次减少组件的 render，却能给性能带来成倍的提升，如 Multiple Connect 和 Smart Seat 方案中，减少 Seats 组件的渲染带来了 5x 的提升。Canvas Seat 方案中，减少了 999 个组件的渲染（1 个 canvas 代替 1000 个 Seat ），带来 10x 的性能提升！

而且，另一个让我感受很深的就是，除了 Multiple Connect 方案是根据 Dan 神提出的思想实现的，其余 2 个方案都是我们手Q队员提出的，而且性能都比 Dan 神的方案好！这让我学习到，作为一个程序员，能因地制宜，发挥自己的 _想象力_ 提出更针对性的方案更重要
