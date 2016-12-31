---
title: 原型继承
date: 2016-12-31 11:51:01
categories: 专业技能
tags:
  - 面向对象
---

简介：学习原型继承的原理
<!-- more -->
## 目录
* 术语
* 原型继承的使用场景
* 原型继承的使用方法
* 原理

## 术语
1. Constructor：泛指构造函数
2. Class：泛指类
3. instance：泛指实例
4. 原型对象：prototype属性的值

## 使用场景
想要使对象拥有 **新属性** ，而该属性是在其原型链上的
_若不介意是在其原型链上，还是在其本身，则为该对象赋值即可_

## 使用方法
1. 用new关键字调用构造函数，使实例原型继承该构造函数
```javascript
function Person() {}
Person.prototype.sayHi = function (){}
// person原型继承Person
const person = new Person()
console.log(person.sayHi)
```

2. 用extends关键字，使一个类原型继承另一个类
```javascript
class Person {
    sayHi() {}
}
// Student原型继承Person
class Student extends Person {
    study() {}
}
// student原型继承Student
const student = new Student()
console.log(student.sayHi)
console.log(student.study)
```

## 原理
在了解原理前，先了解2个关键的知识点：prototyoe属性，和__proto__属性

### prototype属性
prototype属性**只存在于构造函数的属性中**，其值为一个对象，该对象用于储存其所有实例**共享**的属性和方法。（以后称该对象为原型对象）

类比：只有一代宗师（构造函数），才有用于记录自己招式的武功秘籍（原型对象），而其所有徒弟（实例），都能访问到武功秘籍上的招式（属性）

### __proto__属性
__proto__属性存在于每个对象中，值为该对象的原型
_好吧，其实null对象没有__proto__属性，但咱先忽略_

类比：每个人（对象），都有一条链子（__proto__）。链子的另一端，绑着其师傅（构造函数）的武功秘籍（原型对象）

### 原型继承原理
访问对象的属性时，如果在对象上找不到指定属性，则不断在对象的__proto__上查找，一直到原型链最顶层都没找到，则返回undefined

假设当前实例为instance，在instance上找不到，则会在instance.__proto__上找，再找不到，则在instance.__proto__.__proto__上找，一直到Object.prototype.__proto__也就是null都没找到，则返回undefined

题外话
在JavaScript世界中。构造函数Object**原型继承**null，Function**原型继承**Object，最后Function**实例**出能实例千千万的构造函数function。
```javascript
console.log(Object.prototype.__proto__ === null)
// true, Object继承null
console.log(Function.prototype.__proto__ === Object.prototype)
// true，构造函数Function继承Object
function P(){}
console.log(P.__proto__ === Function.prototype)
// true，构造函数P，是构造函数Function的实例
const p = new P()
console.log(p.__proto__ === P.prototype)
// true，实例p，是构造函数P的实例
```

#### 实例原型继承构造函数的原理
在使用new关键字调用构造函数时，JS自动把实例的__proto__属性指向构造函数的prototype对象
```javasript
function Person() {}
const person = new Person()
// 等价于
function Person() {
    // this.__proto__ = Person.prototype
    // return this
}
```

#### 构造函数继承构造函数的原理
```javascript
class Student extends Person {}
// 等价于
Student.prototype.__proto__ = Person.prototype
```
