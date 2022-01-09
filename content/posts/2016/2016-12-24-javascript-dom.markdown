---
title: JavaScript DOM 编程艺术 读书笔记
author: Young
date: 2016-12-24
tags:
  - javascript
  - 计算机
  - dom
  - 读书笔记
category:
  - technology
---

# 读后感
第一次认真看完前端的书籍，理解到了网页上的“MVC”，即：结构，样式，行为。

## Model
模型，在此处即结构，DOM树，网页的核心内容，一个网页，可以没有行为，也可以没有样式，但是一定得有结构；在html结构中，可以指定样式，也可以嵌入行为。

## View
样式，网页的外观等，大多用CSS来实现；

## Controller
行为，最灵活的一个部分，让网页上的一切“动”起来的灵魂，大多用JavaScript来实现。在JavaScript中，可以修改结构，也可以修改样式。

写网站的时候，一定要注意的一点就是：平稳退化。


# 摘抄笔记

这本书的重点是JavaScript，笔记中如果没有特别描述，都是在讲JavaScript.

* 全局变量，可以在脚本的任何位置被引用。
* var关键字可以明确的为函数变量设定作用域。如果在某个函数中使用了var，那么变量就将被视为一个局部变量，反之，如果没有使用var，那个变量就将被视为一个全局变量。
* 在定义一个函数时，我们一定要把它内部的变量全部明确地声明为局部变量。总是使用var关键字，就能避免任何形式的二义性隐患
* DOM===Document Object Model
* JavaScript里的对象可以分为三种类型。
 * 用户定义对象：程序员自己创建的。
 * 内建对象：内建在JavaScript语言里的对象，Array,Math,Date等
 * 宿主对象：由浏览器提供的对象。例如：window
* DOM的原子是元素节点
* 元素节点的nodeType === 1
* 属性节点的nodeType === 2
* 文本节点的nodeType === 3

```html
<p> id='x'>Hello Motor!</p>
```

* 包含在p元素里的文本是另一种节点，他是p元素的第一个子节点。文本的内容可以用alert(x.childNodes[0].nodeValue);显示出来
* 如果使用JavaScript，就要确认，这么做会对用户的浏览体验产生怎样的影响，还有个更重要的问题，如果用户的浏览器不支持JavaScript该怎么办？所谓的平稳退化，就是虽然某些功能无法使用，但是最基本的操作仍能顺利完成
* 接上一条，把链接类节点里，href属性设置为真实存在的URL地址后，即使JavaScript被禁用，这个链接也是可用的，这是一个经典的平稳退化的例子
* ```if(!document.getElementsByTagName) return false```,具有良好的向后兼容性。
* 出于可读性的考虑，把return false的语句尽量全部集中到函数的开头部分。
* 共享onload事件

```javascript
function addLoadEvent(func){
  var oldonload = window.onload;
  if (typeof window.onload != 'function'){
    window.onload =func;
  }else{
    window.onload = function(){
      oldonload();
      func();
    }
  }  
}
```

* 千万不要忘记并非所有的用户都使用鼠标。比如视力残疾的用户往往无法看清屏幕，更喜欢使用键盘
* 结构与行为的分离程度，越大越好
* JavaScript通过创建新元素和修改现有元素来改变网页结构
* 把结构，行为和样式分开永远都是一个好主意，避免在```<body>```部分乱用```<script>```标签，避免使用document.write方法
* createElement, appendChild, CreateTextNode
* insertBefore(), parentNode
* 利用nextSibling(下一个兄弟元素)来做一个自己的insertAfter()
* Ajax
 * 异步加载页面内容，Ajax的主要优势就是对页面的请求以异步方式发送到服务器
 * XMLHttpRequest对象由许多的方法，其中最有用的是open方法，它用来指定服务器上将要访问的文件，指定请求类型:GET,POST,SEND
 * onreadystatechange是一个事件处理函数
 * 访问服务器发送回来的数据要通过两个属性完成，一个是responseText属性，这个属性用于保存文本字符串形式的数据。另外一个属性是responseXML属性，用于保存Context-Type头部中指定为text/xml的数据，其实是一个DocumentFragment对象。
 * 要构建成功的Ajax应用，关键在于将Ajax功能看做一般的JavaScript增强功能，在平稳退化的基础上求得渐进增强
 * 用Ajax拦住发送到服务器的请求，并将请求转交给XMLHttpRequest对象处理。这种情况之下，Ajax就扮演了一个位于常规站点之上的层。
 * 构建Ajax网站的最好方法，也是先构建一个常规的网站，然后Hijix它，渐进增强的使用Ajax。
* Progressive enhancement,内容应该在刚开始编写文档时就成为文档的组成部分
* 平稳退化。
* 有的浏览器会把换行符解释为一个文本节点。
* 在某一类场合，需要决定采用纯粹的CSS来解决还是利用DOM来设置样式，需要考虑一下因素：这个问题最简单的解决方案是什么，哪种解决方案会得到更多浏览器的支持
* 如果你引用一个外部css样式表，并且其中有一条针对.intro类的样式声明, 想要把元素的样式改成这种，那把属性设置为intro就可以达到目的。

```javascript
.intro{
  font-weight: bold;
  font-size: 1.2em;
}

elem.setAttribute("class", "intro");
``` 
* 在需要给一个元素添加class属性时，该怎么做

