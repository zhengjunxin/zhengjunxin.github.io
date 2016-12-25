---
title: 构造函数
date: 2016-12-25 20:45:43
categories: 专业技能
tags:
  - 面向对象
---

简介：本文介绍了构造函数，构造函数的prototype属性，构造函数的prototype.constructor属性的应用场景和使用方式。在ES6新增的类的对应使用方式
<!-- more -->
### 术语
* Constructor：构造函数
* instance：实例

### 目录
* 构造函数
* prototype属性
* constructor属性
* 类
* 总结
* 自测题目

### 构造函数:Constructor
JavaScript中用`new`关键字调用的就是构造函数
```javascript
function Person(age) {
    this.age = age
    this.say = function() {
        // 实例能访问this.age，this就是实例自己
          console.log(`My age is ${this.age}`)
    }
}
const p = new Person(18)
p.say()
```
* 原理：使用`new`调用构造函数时，其函数体定义的`this`会默认返回作为实例
* 应用场景：定义**私有**属性和方法
* 使用方式：在构造函数的函数体内的`this`定义。如上所示，为`this`定义`age`属性和`say`方法，即可使其实例拥有对应的私有属性和方法

### prototype属性：Constructor.prototype
**每个构造函数都有一个prototype属性**，其值为一个对象（以后称prototype所指的对象为**原型对象**）
> 原型对象的用途，是储存其所有实例**共享**的属性和方法。《JavaScript高级程序设计》P147  

```javascript
function Person() {}
Person.prototype.age = 18
Person.prototype.say = function() {
    // 实例能访问this.age，是因为age是实例的原型属性，在其原型链上
    console.log(`My age is ${this.age}`)
}
const p = new Person()
p.say()
```
* 原理：实例继承构造函数的原型对象，所以能通过原型链访问
* 应用场景：定义**共享**属性和方法
* 使用方式：在构造函数的原型对象上定义。如上所示，为`Person.prototype`定义`age`属性和`say`方法，即可使所有实例都拥有对应的共有属性和方法

### constructor属性：Constructor.prototype.constructor
> 所有原型对象都会自动获得一个constructor属性，指向prototype属性所在的构造函数。《JavaScript高级程序设计》P148  

```javascript
function Person() {}
Person.age = 18
Person.say = function() {
    // 注意，此时还能用this，访问this.age
    console.log(`My age is ${this.age}`)
}
const p = new Person()
p.constructor.say()
```
* 原理：构造函数也是对象，能定义自己的属性和方法（称为静态属性和方法）
* 应用场景：定义默认属性，如React的defaultProps。和**校验方法**如`Array.isArray`
* 使用方式：直接在构造函数这个对象上定义。如上所示`Person.name`，然后调用时通过`p.constuctor.say`。因为**实例是继承构造函数的原型对象，而不是继承构造函数**

### 类
ES6新增了class关键字，允许我们定义类。但其实class只是**构造函数的语法糖**而已
```javascript
class Person {
    // 等价于Person.staticSay = function(){}
    static staticSay() {}
    constructor(age) {
        this.age = age
        // 等价于this.instanceSay = function(){}
        this.instanceSay = function(){}
    }
    // 等价于Person.prototype.prototypeSay = function(){}
    prototypeSay() {}
}
const p = new Person()
```
其中加了`static`关键字的方法，相当于构造函数的静态方法，`constructor`相当于构造函数的函数体，而非`constructor`的其余方法，则相当于构造函数的原型方法

### 总结
1. Constructor的函数体是定义实例的**实例属性和方法**的
2. Constructor.prototype是定义实例的**共享属性和方法**的
3. Constructor上是定义构造器的**静态属性和方法**的
4. 类知识构造函数的语法糖

### 自测题目
1. 声明一个构造函数Student，使得该构造函数产生的实例
    1. 有**私有**的`name`属性和`study`方法
    2. 有**共有**的`school`属性和`say`方法
    3. 有静态属性`defaultProps`和用于校验是否`Student`实例的`isStudent`方法
2. 使用类完成上述功能

#### 答案
```javascript
function Student(name) {
    this.name = name
    this.study = function() {
        console.log('unique study method')
    }
}
Student.prototype.shcool = 'MIT'
Student.prototype.say = function() {
    console.log('common say method')
}
Student.defaultProps = 'student'
Student.isStudent = function(instance) {
    return instance instanceof Student
}
const joe = new Student('joe')

console.log(joe.name) // joe
joe.study() // unique study method
console.log(joe.shcool) // MIT
joe.say() // common say method
console.log(Student.isStudent(joe)) // true
```
