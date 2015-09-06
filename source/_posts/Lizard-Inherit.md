title: 《Lizard源码分析》之Common组件Core模块［一］
date: 2015-08-29 15:51:39
categories: [御剑江湖, Javascript]
tags: [Javascript, Lizard]
---
![](/img/normal/prototype.jpg)
在职业生涯中长久保持对技术的热忱是非常重要的事，在使用业务**API**的时候往往忽略了它的核心思想。今天便好好探索下**Lizard**框架中核心模块**Core**对抽象类的实现。

抽象类的作用就是为了创建实例和继承用的，谈及继承，大家都知道**Javascript**初级版本对继承的支持主要是依赖于**prototype**原型链实现的，而**ES6**已经把**Class**与**entends**的概念真正的实现到了服务端语言的层次，不得不令人感慨，真此一时彼一时也。

###言归正传，打开**Lizard**源码**c.class.inherit.js**：
```
var slice = [].slice; // 抽离数组的slice函数，转型伪数组的通用手法
// 核心构造函数
var Core = function () {

};
```
之前一直没有过直接给函数添加属性或方法的习惯，但事实上**function**本身也是**object**类型，可以任意扩展。在构造函数场景中，附加的属性和方法代表实例拥有的特性与行为，而**jQuery**特有**$**函数上扩展的各个方法追加进了各个通过**selector**返回的**jQuery**类型**dom**对象之中，并实现了链式调用。

**Core**的核心方法`Class`，我们先看它的完整代码：
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
###整个Class函数体可分为四个段落：

####1.［变量初始化］
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
当参数限制后，`Class`的作用限制在要么通过反射创建一类对象的实例，要么就通过一个实例作为父类来扩展此对象。`properties`永远装载的是待扩展对象的属性集，以对象表示。

####2.［虚拟构造函数］
```
/**
 * 虚拟构造函数 构造器
 * 寄生组合继承的用法[1]
 */
function klass() {
    this.__propertys__();// 执行私有的__propertys__函数
    this.initialize.apply(this, arguments);// 执行自定义构造函数
}
// 虚拟构造函数superclass属性保存父类实例 运行时为构造函数或为空
klass.superclass = parent; 
// 虚拟构造函数subclasses属性保存子类变量 创建空数组 装载多个子类来一一继承父类
klass.subclasses = []; 

var sup__propertys__ = function () {};
var sub__propertys__ = properties.__propertys__ || function () {};

if (parent) {

    if (parent.prototype.__propertys__) {
    	// 如果父类原型中包含了一个__propertys__属性扩展函数则赋值给sup__propertys__
    	sup__propertys__ = parent.prototype.__propertys__;
    }
    var subclass = function () {};
    subclass.prototype = parent.prototype; 
    klass.prototype = new subclass; 

    parent.subclasses.push(klass);// ?? parent.subclasses.push(subclass);
}
```
所有的对象创建均为虚拟构造函数`klass`的实例，并作为`Core.Class`的返回值返回。
当父类不为空的时候，会在父类的基础上做原型的扩展，而子类的构造函数`subclass`跟父类实例的原型对象保持一致，并使虚拟构造函数`klass`的原型指向子类`subclass`的实例。
在此有个疑问，作者为何给`parent.subclasses`数组中放置的不是`subclass`而是`klass`？为`klass`时，原型链会有个循环引用的指向，`klass`不停包裹着`klass`，最后这句似乎改成`parent.subclasses.push(subclass)`更加合理……

####3.［实例扩展与子类重写］
```
// 作为父类原型对象 可为空
var ancestor = klass.superclass && klass.superclass.prototype;
// for－in循环遍历属性对象
for (var k in properties) {
    // 取出每个属性    
    var value = properties[k];
	// 如果父类实例与对象中的属性为fn
    if (ancestor && typeof value == 'function') {
    	// 强大又很复杂的正则 匹配函数中所有参数名
        // 此处也一定是class函数的精华部分
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
    // 属性赋值给虚拟klass
    klass.prototype[k] = value;
}
```
最好玩的部分来了，**"ancestor"**这个单词的含义是祖宗抑或原型，而实际上装载的也就是想当实例它祖宗的原型。
当待扩展属性对象`properties`包含一个`function ($super, options) {}`的函数时，只要匹配到了`$super`，便重置子类参数数组并合并，可以交由父类来调用。
最后把每个属性都扩展到父类的原型对象上。

####4.［扩展与兼容］
```
// 如果虚拟的原型中不包含initialize构造函数 则创建空函数做为虚拟构造函数中要调用的真实构造函数
if (!klass.prototype.initialize) klass.prototype.initialize = function () {};
// 兼容现有框架，__propertys__方法直接重写		
klass.prototype.__propertys__ = function () {
    // 父类和子类的扩展属性函数都作为虚拟函数来执行一遍
    // 无父类继承状态下 和 无私有__propertys__的属性函数下均执行的是空函数
    sup__propertys__.call(this);
    sub__propertys__.call(this);
};

for (key in parent) {
    if (parent.hasOwnProperty(key) && key !== 'prototype' && key !== 'superclass')
    klass[key] = parent[key];
}

klass.prototype.constructor = klass;
// 返回klass虚拟构造函数
return klass;
```
`initialize`函数的设定是作为实例的构造函数重写存在的，而默认都会给一个无参的函数以供调用。
`__propertys__`函数的设定是作为属性扩展函数使用的，其中可封装一定的业务逻辑。
在`parent`父类实例中，是自己独立属性的情况下（非原型也非`superclass`）也会兼容扩展给虚拟构造函数`klass`。

好了，大致逻辑分析到一段落，下面看个小demo：

```
var Animal = Core.Class({
    __propertys__:function () {
        this.move = function () {
            console.log('The Animal is moving.');
        }
    },
    // 自定义构造函数 options
    initialize: function (type, age) {
        this.type = type;
        this.age = age;
    },
    eat: function (food) {
        food = food || '肉';
        console.log('我要吃' + food + '~~~~');
    },
    showSelf: function () {
        console.log('Type: ' + this.type + '; Age: ' + this.age + '; ');
    }
});

var Cat = Core.Class(Animal, {
    __propertys__:function () {
        this.name = 'Mimi';
    },
    initialize: function ($super, type, age) {
        // 运行时使用实际传递参数，与父类中参数保持一致
        // 这点很重要 当参数中包含$super要避免漏参
        $super(type, age);
    },
    showSelf: function () {
        console.log('Name: ' + this.name + '; Type: ' + this.type + '; Age: ' + this.age + '; ');
    }
});

var animal = new Animal('Cat', '2');
animal.showSelf(); // 输出：Type: Cat; Age: 2;

var cat = new Cat('Cat', '3');
cat.eat('金坷垃'); // 调用父类方法 输出：我要吃金坷垃~~~~
cat.showSelf(); // 输出：Name: Mimi; Type: Cat; Age: 3; 
```
前面源码中会对`$super`参数做处理，若子类含有`$super`参数，那么如果父类含有同名方法，则可调用父类方法，可控制权在子类手里，这里唯一要注意的就是参数传递，如demo中注释一定要一一对应。

整个Core.Class函数分析到此结束，给自己也重新温习了下基础知识，后面封装代码尽量使扩展性更佳更友好！

@^_^@


