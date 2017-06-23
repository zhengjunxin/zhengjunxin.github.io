---
title: 为什么 React 需要 Redux 这类状态管理库
date: 2017-06-22 21:05:01
tags:
  - React
---

简介：用生动形象的例子总结为什么 React 需要 Redux 这类状态管理库

<!--more -->

在开始用 Redux 时，只知道 Redux 是用来管理状态的，很吊，大家都用他，所以就用了。对于状态指的是什么？为什么需要 Redux 来管理状态？其实我并不是很清楚。
现在空闲下来，所以思考一下为什么 React 需要 Redux 这类状态管理库呢？我个人总结出的答案是为了*方便组件间的通信*，在 React 中，父组件向子组件通信很容易，但对祖父向孙子，子向父，兄弟组件间的通信则比较麻烦。而在大型的应用，这 3 种场景是无法避免的，所以需要借助 Redux 来解决这个问题。且容我用一个生动形象的例子解释一下。

## 父组件与子组件通信

假设我们需要实现一个类似[购物车页面](https://joezheng2015.github.io/learningReact/#/why_redux)，我们的代码大概会是这样的

``` javascript
const Goods = ({ goods, increment, decrement }) => (
    <div>
        {
            goods.map(good => (
                <div>
                    {good.name} {good.number}
                    <button onClick={() => increment(good.id)}>+</button>
                    <button onClick={() => decrement(good.id)}>-</button>
                </div>
            ))
        }
    </div>
)

class Shop extends React.Component {
    state = {
        goods: [
            { id: 0, name: 'iphone', number: 0 },
            { id: 1, name: 'mbp', number: 0 },
        ]
    }
    render() {
        return (
            <div className="App">
                <Goods goods={this.state.goods} increment={this.increment} decrement={this.decrement} />
            </div>
        )
    }
    increment = (goodId) => {
        this.updateGoods(goodId, true)
    }
    decrement = (goodId) => {
        this.updateGoods(goodId, false)
    }
    updateGoods(goodId, isIncrement) {
        this.setState({
            goods: this.state.goods.map(good => {
                if (good.id === goodId) {
                    return Object.assign({}, good, {
                        number: isIncrement ? good.number + 1 : good.number - 1,
                    })
                }
                return good
            })
        })
    }
}
```

Shop 组件负责维护数据 goods 和对应的更新方法 increment 和 decrement，而 Goods 组件负责展示商品，所以只需要 Shop 把 goods 和 increment, decrement 传递给其子组件 Goods 即可，即父组件 Shop 通过传递 props 给子组件 Goods，实现父子组件间的通信。但是随着功能的增多，我们可能需要把商品单独分出一个组件来，这样一来我们就需要实现祖父组件与孙子组件间的通信了

## 祖父组件与孙子组件间的通信

``` javascript
const Good = ({ good, increment, decrement }) => (
    <div>
        {good.name} {good.number}
        <button onClick={() => increment(good.id)}>+</button>
        <button onClick={() => decrement(good.id)}>-</button>
    </div>
)
const Goods = ({ goods, increment, decrement }) => (
    <div>
        {
            goods.map(good => <Good key={good.id} good={good} increment={increment} decrement={decrement} />)
        }
    </div>
)

class Shop extends React.Component {
    state = {
        goods: [
            { id: 0, name: 'iphone', number: 0 },
            { id: 1, name: 'mbp', number: 0 },
        ]
    }
    render() {
        return (
            <div className="App">
                <Goods goods={this.state.goods} increment={this.increment} decrement={this.decrement} />
            </div>
        )
    }
    increment = (goodId) => {
        this.updateGoods(goodId, true)
    }
    decrement = (goodId) => {
        this.updateGoods(goodId, false)
    }
    updateGoods(goodId, isIncrement) {
        this.setState({
            goods: this.state.goods.map(good => {
                if (good.id === goodId) {
                    return Object.assign({}, good, {
                        number: isIncrement ? good.number + 1 : good.number - 1,
                    })
                }
                return good
            })
        })
    }
}
```

此时就需要把 goods 和 increment, decrement，从祖父组件 Shop 传递给父组件 Goods，再传递给 Good。这时就能发现，这样传递有点麻烦了，不过还在可接受范围内。但是，随着规模的增加，说不定会出现曾曾祖父组件与其曾曾孙组件的通信，这样层层传递数据就会显得很头疼。

## Redux 的通信方式

借助 Redux 这类状态管理库，通过定义 store，在需要的组件调用 connect 即可访问指定的数据（如 goods ）和对应的更新数据的 dispatch 方法。只要简单的一个 connect，无论是曾孙组件还是玄孙组件，都能马上访问到数据，免去层层传递的麻烦！

``` javascript
export default connect(state => ({ goods: state.goods }))(Goods)
```

## 总结
状态管理库中所谓的*状态*，即用于渲染视图的数据。而 React 需要 Redux, Mobx 这类状态管理库，是为了*更方便组件间的通信*。无论组件的层级关系是祖父与孙子、曾祖父与曾孙子，只需要 connect 即可获取数据，和更新数据的方法，而不用层层传递。

