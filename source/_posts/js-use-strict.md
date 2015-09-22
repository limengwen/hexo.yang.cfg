title: 《回首Javascript基础》之严格模式
date: 2015-09-17 23:06:19
categories: [御剑江湖, JavaScript]
tags: [JavaScript]
---
![](/img/game/call_on_duty.jpg)
写了那么久的业务代码，记着了该记着的又常用的，但感觉好记性真的不如烂笔头，重复造轮子是必经之路，但不断夯实基础才是飞升之路的重中之重。

`use strict`现在业务场景中用到的越来越多，目的也是为了让Js写得更加规范严谨，Js本身是弱类型语言，编写脚本的时候如果控制的不够，那么就很容易出现安全漏洞，比如变量未声明变成了全局变量，然后跑在了各个函数里使用，并且被重名变量更近一步的改写，导致出现错误值，bug藏的异常的深，现在突然想起填过的深坑，所以写出安全的Js脚本是有多么的重要，那么这里再次做个详细的总结。


###一、添加部署
`Warning ： Strict mode is not supported in versions of Internet Explorer earlier than Internet Explorer 10.`
这段是MSDN上的第一个警告就标明，IE10以下的版本是不兼容的，而我们当今的项目基本都是WebApp了，所以跟进严格是非常必要的，而不支持的版本也仅仅是当成在内存里丢了一个‘use strict’的字符串，并无任何问题。

1、放置在脚本文件头部
```
"use strict";
function testFunction(){
    var testvar = 4;
    return testvar;
}

// 这里会语法报错
testvar = 5;
```
这里需要注意的是，如果有多个脚本合并压缩的时候，此部署方法会把后面所有的脚本也变成严格模式，产生不必要的麻烦。

2、放置在函数体内部
```
function testFunction(){
    "use strict";
    // 这里会语法报错
    testvar = 4;
    return testvar;
}
testvar = 5;
```
此部署方法在合并压缩就不会产生上述问题，严格模式只针对`testFn`函数有效。

###二、约束内容

1、错误拼写
```
"use strict";
                       // 假如有一个全局变量叫做mistypedVariable
mistypedVaraible = 17; // 因为变量名拼写错误
                       // 这一行代码就会抛出 ReferenceError
```
针对全局变量的控制，严格模式已经不允许意外的创建它。

2、只读与不可扩展对象约束
```
"use strict";

// 给不可写属性赋值
var obj1 = {};
Object.defineProperty(obj1, "x", { value: 42, writable: false });
obj1.x = 9; // 抛出TypeError错误

// 给只读属性赋值
var obj2 = { get x() { return 17; } };
obj2.x = 5; // 抛出TypeError错误

// 给不可扩展对象的新属性赋值
var fixed = {};
Object.preventExtensions(fixed);
fixed.newProp = "ohai"; // 抛出TypeError错误
```
正常模式下是静默失败，而严格模式会直接报错。

3、不可删除原生对象或其属性
```
"use strict";
delete Object.prototype;
```


想想当初Java搞了一年多，现在也应该重新拣起来才对，蓦然回首，还是初来魔都时更加勤奋，连上下班的地铁时间也不敢浪费。如今，一样要好好继续给自己加油。

（PS.放些《使命召唤》的图片似乎更能表现“严格”的重要性。）

参考文档：
[MSDN《Strict Mode (JavaScript)》](https://msdn.microsoft.com/en-us/library/br230269.aspx)
[Mozilla 《严格模式》](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)
[阮一峰老师《Javascript 严格模式详解》](http://www.ruanyifeng.com/blog/2013/01/javascript_strict_mode.html)

