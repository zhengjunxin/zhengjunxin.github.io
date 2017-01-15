---
title: npm模块开发问题总结
date: 2017-01-15 15:56:43
categories: 专业技能
tags:
---
简介：总结在开发 npm 模块过程遇到的一些问题
<!-- more -->

## 目录
* LICENSE
* 导出模块
* 命名冲突

## LICENSE
在初始化 npm 时需要指定开源协议，当时还各种找模板，其实在 github 创建新的仓库时就可以指定生成了，也有 简单的[介绍](http://choosealicense.com/)

{% img /images/license.jpg %}

## 导出模块
在导出 npm 模块时，一开始我很天真的使用 ES6 的语法（还没被实现的语法，我怎么想的，真是笑死我了）
```javascript
export default myFunction
```

再了解到 Webpack 同时支持 CommonJS 和 AMD 方案后，决定使用 AMD 方案。结果 [create-react-app](https://github.com/facebookincubator/create-react-app) 又不支持 AMD 所以最后使用了 [UMD](https://github.com/umdjs/umd) 的 [returnExports](https://github.com/umdjs/umd/blob/master/templates/returnExports.js) 兼 Node, AMD 和 browser globals 3 种使用方式，顺便整理了 JS 模块的规范：

### AMD
RequireJS使用的方案
```javascript
// define暴露
define(function() {
    // 返回想要暴露的对象
    return objectYourWantToExport
})

// require引入
require([module], callback)
```

### CommonJS
NodeJS使用的方案
```javascript
// exports暴露
module.exports = jquery

// require引入
var $ = require('jquery')
```

### UMD
[UMD github](https://github.com/umdjs/umd) 通用模块规范用于适配各种场景

## 命名冲突
{% img /images/permission.jpg %}

在发布 npm 模块时出现 permission 错误，还以为是哪一步错了，结果是包的命名冲突了，更改 package.json 中的 name 即可