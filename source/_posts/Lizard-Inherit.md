title: 《Lizard源码分析》之Class继承
date: 2015-08-29 15:51:39
categories: [御剑江湖, Javascript]
tags: [Javascript, Lizard]
---
在职业生涯中长久保持对技术的热忱是非常重要的事，在使用业务API的时候往往忽略了它的核心思想。今天便好好探索下Lizard框架对继承的实现。

谈起继承，大家都知道Javascript语言初级版本对继承的支持主要是依赖于prototype原型链实现的，而ES6已经把Class与entends的概念真正的实现到了服务端语言的层次，不得不令人感慨，真此一时彼一时也。

言归正传，打开Lizard源码c.class.inherit.js：

``` bash
var slice = [].slice;
var Core = function () {

};
```

``` bash
Core.Class = function () {
if (arguments.length == 0 || arguments.length > 2) throw '参数错误';

var parent = null;
//将参数转换为数组
var properties = slice.call(arguments);

//如果第一个参数为类（function），那么就将之取出
if (typeof properties[0] === 'function')
parent = properties.shift(); // 父类构造函数
properties = properties[0]; // properties 第一个

function klass() {
this.__propertys__();
this.initialize.apply(this, arguments);
}

klass.superclass = parent; // 父类 构造函数 || 空
klass.subclasses = []; // 子类s
// 父类属性
var sup__propertys__ = function () {
};
// 子类属性 函数
var sub__propertys__ = properties.__propertys__ || function () {
};

if (parent) {
if (parent.prototype.__propertys__) sup__propertys__ = parent.prototype.__propertys__;

var subclass = function () {
};
subclass.prototype = parent.prototype;
klass.prototype = new subclass;
parent.subclasses.push(klass);
}


var ancestor = klass.superclass && klass.superclass.prototype;
for (var k in properties) {
var value = properties[k];

//满足条件就重写
if (ancestor && typeof value == 'function') {
var argslist = /^\s*function\s*\(([^\(\)]*?)\)\s*?\{/i.exec(value.toString())[1].replace(/\s/i, '').split(',');
//只有在第一个参数为$super情况下才需要处理（是否具有重复方法需要用户自己决定）
if (argslist[0] === '$super' && ancestor[k]) {
value = (function (methodName, fn) {
return function () {
var scope = this;
var args = [function () {
return ancestor[methodName].apply(scope, arguments);
} ];
return fn.apply(this, args.concat(slice.call(arguments)));
};
})(k, value);
}
}

klass.prototype[k] = value;
}

if (!klass.prototype.initialize)
klass.prototype.initialize = function () {
};

//兼容现有框架，__propertys__方法直接重写
klass.prototype.__propertys__ = function () {
sup__propertys__.call(this);
sub__propertys__.call(this);
};

//   //兼容代码，非原型属性也需要进行继承
//   for (key in parent) {
//     if (parent.hasOwnProperty(key) && key !== 'prototype')
//       klass[key] = parent[key];
//   }

//兼容代码，非原型属性也需要进行继承
for (key in parent) {
if (parent.hasOwnProperty(key) && key !== 'prototype' && key !== 'superclass')
klass[key] = parent[key];
}

klass.prototype.constructor = klass;

return klass;
};


/**
* @description 对象扩展
* @method Common.cCoreInherit.extend
* @param {object} targetObj 原型对象
* @param {object} sourceObj1 要继承的对象1
* @param {object} ... 要继承的对象2
* @returns {object} res
* @example
*  var O1 = {
*    "name" : "jim"
*  };
*
*  var O2 = {
*    "age": 18
*  }
*
*  var O3 = {};
*  cCoreInherit.extend(O3,O1,O2)
*  // O3 = {
*    "name" : "jim",
*    "age": 18
*  }
*/
Core.extend = function () {
var args = slice.call(arguments);
var source = args.shift() || {};

if (!source) return false;

for (var i = 0, l = args.length; i < l; i++) {
if (typeof args[i] === 'object') {
for (var key in args[i]) {
source[key] = args[i][key];
}
}
}

return source;
};

/**
* @description 对原型链的扩充
* @method Common.cCoreInherit.implement
* @param {function} fn 构造函数
* @param {object} propertys 需要補充在原型链上的方法和属性
* @returns {Function}
*/
Core.implement = function (fn, propertys) {
if (typeof fn !== 'function') return false;

for (var i in propertys) {
fn.prototype[i] = propertys[i];
}

return fn;
};

return Core;

```
