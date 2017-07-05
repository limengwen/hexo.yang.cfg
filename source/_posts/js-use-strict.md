title: 《回首Javascript基础》之严格模式
date: 2015-09-17 23:06:19
categories: [御剑江湖, JavaScript]
tags: [JavaScript]
---
![使命召唤——use strict](/img/game/call_on_duty.jpg)
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
delete Object.prototype; // 抛出TypeError错误
```

4、属性名必须唯一
```
"use strict";
var o = { p: 1, p: 2 }; // !!! 语法错误
```
ECMAScript6修复了此问题。

5、参数名必须唯一
```
function sum(a, a, c){ // !!! 语法错误
  "use strict";
  return a + b + c; // 代码运行到这里会出错
}
```

###三、简化变量
1、禁用`with`
2、禁止删除声明变量
3、禁止随意控制eval和arguments,不允许为其二者赋值。
```
"use strict";
// 以下均会报错
eval = 17;
arguments++;
++eval;
var obj = { set p(arguments) { } };
var eval;
try { } catch (arguments) { }
function x(eval) { }
function arguments() { }
var y = function eval() { };
var f = new Function("arguments", "'use strict'; return 17;");
```
4、为arguments解锁
```
function f(a)
{
  "use strict";
  a = 42;
  return [a, arguments[0]];
}
var pair = f(17);
console.assert(pair[0] === 42); // Success
console.assert(pair[1] === 17); // Success
```
改变传递参数并赋值并不会与arguments产生关联关系，而正常模式下会。平时编码中我们很可能会不小心踩此坑。

5、不再支持 arguments.callee

###四、消除安全隐患
1、this的权限
```
"use strict";
function fun() { return this; }
assert(fun() === undefined);
assert(fun.call(2) === 2);
assert(fun.apply(null) === null);
assert(fun.call(undefined) === undefined);
assert(fun.bind(true)() === true);
```
`this`终于可以被释放，不再总是指向一个对象，默认总是指向当前对象，如果都是指向window对象肯定会不小心就埋了深坑。

2、关键字的取缔
```
function package(protected) // !!!
{
  "use strict";
  var implements; // !!!

  interface: // !!!
  while (true)
  {
    break interface; // !!!
  }

  function private() { } // !!!
}
function fun(static) { 'use strict'; } // !!!
```
这里收归国有也是为了将来代码可以更好的支持ECMAScript6。

`strict mode`先把常用要注意的总结到这里，mark了不少Mozilla的内容。可能相信在这些约束下，Js可以变的越来越规范，很多同行可能会觉得如果去掉了这些“灵活”的内容，Js就不好玩了，我个人觉得恰恰相反，因为Js开发成本低但它的排障成本比其他任何语言都高很多，所以我们的代码需要去走“严格模式”。

大家一起“use strict”起来吧！


参考文档：
[MSDN《Strict Mode (JavaScript)》](https://msdn.microsoft.com/en-us/library/br230269.aspx)
[Mozilla 《严格模式》](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)
[阮一峰老师《Javascript 严格模式详解》](http://www.ruanyifeng.com/blog/2013/01/javascript_strict_mode.html)
