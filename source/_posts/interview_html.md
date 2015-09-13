title: 《前端十万个为什么》之HTML卷
date: 2015-09-13 14:11:58
categories: [御剑江湖, Web]
tags: [面试习题, Html]
---
![](/img/tech/i_html_0.jpg)
今天顺手把HTML的内容也整理一点放进来，表述性的内容尽量也都把答案贴到下方以供参考。

**update log：**
`v0.0.1 2015-09-13`
`v0.0.2 2015-09-14`

![](/img/tech/i_html_2.jpg)

###Unclass
**1.Doctype是什么？HTML5的doctype？答：**
`<!DOCTYPE> 声明位于文档中的最前面的位置，处于 <html> 标签之前。此标签可告知浏览器文档使用哪种 HTML 或 XHTML 规范。（重点：告诉浏览器按照何种规范解析页面）。`
`HTML5: <!DOCTYPE html>`

**2.请描述一下cookies，sessionStorage和localStorage的区别？答：**
`sessionStorage用于本地存储一个会话（session）中的数据，这些数据只有在同一个会话中的页面才能访问并且当会话结束后数据也随之销毁。因此sessionStorage不是一种持久化的本地存储，仅仅是会话级别的存储。`
`localStorage用于持久化的本地存储，除非主动删除数据，否则数据是永远不会过期的。`

`Web storage和cookie的区别:`
`<1> Web Storage的概念和cookie相似，区别是它是为了更大容量存储设计的。Cookie的大小是受限的，并且每次你请求一个新的页面的时候Cookie都会被发送过去，这样无形中浪费了带宽，另外cookie还需要指定作用域，不可以跨域调用。`
`<2> 除此之外，Web Storage拥有setItem,getItem,removeItem,clear等方法，不像cookie需要前端开发者自己封装setCookie，getCookie。但是Cookie也是不可以或缺的：Cookie的作用是与服务器进行交互，作为HTTP规范的一部分而存在 ，而Web Storage仅仅是为了在本地“存储”数据而生。`

**3.简述一下src与href的区别。答：**
`src用于替换当前元素，href用于在当前文档和引用资源之间确立联系。`
`src是source的缩写，指向外部资源的位置，指向的内容将会嵌入到文档中当前标签所在位置；在请求src资源时会将其指向的资源下载并应用到文档内，例如js脚本，img图片和frame等元素。`
```
<script src =”js.js”></script>
```
`当浏览器解析到该元素时，会暂停其他资源的下载和处理，直到将该资源加载、编译、执行完毕，图片和框架等元素也如此，类似于将所指向资源嵌入当前标签内。这也是为什么将js脚本放在底部而不是头部。`
`href是Hypertext Reference的缩写，指向网络资源所在位置，建立和当前元素（锚点）或当前文档（链接）之间的链接，如果我们在文档中添加`
```
<link href=”common.css” rel=”stylesheet”/>
```
`那么浏览器会识别该文档为css文件，就会并行下载资源并且不会停止对当前文档的处理。这也是为什么建议使用link方式来加载css，而不是使用@import方式。`

**4.如何理解HTML结构的语义化？**
**5.谈谈以前端角度出发做好SEO需要考虑什么？**

![](/img/tech/i_html_1.jpg)


