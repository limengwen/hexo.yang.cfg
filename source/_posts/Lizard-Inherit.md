title: 《Lizard源码分析》之Class继承
date: 2015-08-29 15:51:39
categories: [御剑江湖, Javascript]
tags: [Javascript, Lizard]
---
在职业生涯中长久保持对技术的热忱是非常重要的事，在使用业务API的时候往往忽略了它的核心思想。今天便好好探索下Lizard框架对抽象类的实现。

抽象类的作用就是为了创建实例和继承用的，谈及继承，大家都知道Javascript初级版本对继承的支持主要是依赖于prototype原型链实现的，而ES6已经把Class与entends的概念真正的实现到了服务端语言的层次，不得不令人感慨，真此一时彼一时也。

言归正传，打开Lizard源码c.class.inherit.js：
```
var slice = [].slice; // 抽离数组的slice函数，转型伪数组的通用手法
// 核心构造函数
var Core = function () {

};
```
之前一直没有过直接给函数添加属性或方法的习惯，但事实上function本身也是object类型，可以任意扩展。在构造函数场景中，附加的属性和方法代表实例拥有的特性与行为，而jQuery特有$函数上扩展的各个方法追加进了各个通过selector返回的jQuery类型dom对象之中，并实现了链式调用。

Core的核心方法Class，我们先看它的完整代码：
```
Core.Class = function () {

    if (arguments.length == 0 || arguments.length > 2) throw '参数错误';
    
    var parent = null; 
    var properties = slice.call(arguments);

    if (typeof properties[0] === 'function') {
        parent = properties.shift(); 
    }
    properties = properties[0]; 


    function klass() {
        this.__propertys__();
        this.initialize.apply(this, arguments); 
    }

    klass.superclass = parent; 
    klass.subclasses = []; 

    var sup__propertys__ = function () {};
    var sub__propertys__ = properties.__propertys__ || function () {};

    if (parent) {

        if (parent.prototype.__propertys__) {
        	sup__propertys__ = parent.prototype.__propertys__;
        }
        var subclass = function () {};
        subclass.prototype = parent.prototype; console.log(parent.prototype);
        klass.prototype = new subclass; 

        parent.subclasses.push(klass);
    }
	
    var ancestor = klass.superclass && klass.superclass.prototype;

    for (var k in properties) {
        
        var value = properties[k];

        if (ancestor && typeof value == 'function') {
            var argslist = /^\s*function\s*\(([^\(\)]*?)\)\s*?\{/i.exec(value.toString())[1].replace(/\s/i, '').split(',');
            if (argslist[0] === '$super' && ancestor[k]) {
            	value = (function (methodName, fn) {
                    return function () {
            	    	var scope = this;
            	    	var args = [function () {
                            return ancestor[methodName].apply(scope, arguments);
                        }];
                        return fn.apply(this, args.concat(slice.call(arguments)));
                    };
                })(k, value);
            }
        }
        klass.prototype[k] = value;
    }

    if (!klass.prototype.initialize) klass.prototype.initialize = function () {};
		
    klass.prototype.__propertys__ = function () {
        sup__propertys__.call(this);
        sub__propertys__.call(this);
    };

    for (key in parent) {
    	if (parent.hasOwnProperty(key) && key !== 'prototype' && key !== 'superclass')
    	klass[key] = parent[key];
    }

    klass.prototype.constructor = klass;

    return klass;
};

```
整个Class函数体可分为四个段落：

1.［变量初始化］
```
// 限制参数 只能有1或2个
if (arguments.length == 0 || arguments.length > 2) throw '参数错误';

var parent = null;// 创建父类变量
var properties = slice.call(arguments);// 将参数转换为数组 并赋值到属性变量中

// 如果第一个参数为类（构造函数），那么就将之取出
if (typeof properties[0] === 'function') {
    // 删除父类构造函数并丢给父类变量
    parent = properties.shift(); 
}

// properties只能是待扩展的自面量对象
properties = properties[0];
```
当参数限制后，Class的作用限制在要么通过反射创建一类对象的实例，要么就通过一个实例作为父类来扩展此对象。properties永远装载的是待扩展对象的属性集，以对象表示。

2.［虚拟构造函数］
```

function klass() {
    this.__propertys__();
    this.initialize.apply(this, arguments); 
}

klass.superclass = parent; 
klass.subclasses = []; 

var sup__propertys__ = function () {};
var sub__propertys__ = properties.__propertys__ || function () {};

if (parent) {

    if (parent.prototype.__propertys__) {
    	sup__propertys__ = parent.prototype.__propertys__;
    }
    var subclass = function () {};
    subclass.prototype = parent.prototype; 
    klass.prototype = new subclass; 

    parent.subclasses.push(klass);// ??
}
```
所有的对象创建均为虚拟构造函数klass的实例，并作为Core.Class的返回值返回。
当父类不为空的时候，会在父类的基础上做原型的扩展，而子类的构造函数subclass跟父类实例的原型对象保持一致，并使虚拟构造函数klass的原型指向子类subclass的实例。
在此有个疑问，作者为何parent.subclasses数组中放置的不是subclass而是klass？为klass时，原型链会有个循环引用的指向，klass不停包裹着klass，最后这句似乎改成parent.subclasses.push(subclass)更加合理……

3.［子类重写］
```
```
最好玩的部分来了
4.［扩展与兼容］
