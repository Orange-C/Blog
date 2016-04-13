---
title: 谈谈JavaScript的作用域链
date: 2016-03-21 19:59:14
tags:
---

## 前言

作用域链，闭包，原型链算是JavaScript中比较有特色的三个知识点。我会分三篇文章讲讲我对于这三个东西的理解。

## JavaScript中的作用域

每一种语言都有作用域的概念，所谓作用域就是变量与函数的可用范围，确定了变量与函数的可见性和生命周期。
JavaScript中有两种作用域，全局作用域和局部作用域

<!-- More -->

### 全局作用域

以下情况一般拥有全局作用域
* 在全局声明的函数和变量都有全局作用域
* 所有未定义直接赋值的变量
* 所有window的属性

### 局部作用域

在函数内部声明的变量和函数只能在该函数内部访问到，即拥有局部作用域

## 作用域链

JavaScript中，一切都是对象，包括函数。而函数有一个内部属性`[[scope]]`，里面包含了函数的可访问的作用域对象的集合，这个集合被称为作用域链。

### 作用域链的创建

作用域链在函数执行时创建。**作用域链的前端，始终是当前环境的变量对象**。如果这个环境是函数，则将其活动对象作为变量对象，里面包含了该函数的所有局部变量，参数以及this。
把作用域链当成一个对象数组，先把该函数的变量对象push进数组，然后把函数的外部环境的作用域push进数组，下一个就是外部环境的外部环境的变量对象，一直往外直到全局环境，**即作用域的最后一个对象肯定是全局环境的作用域**。

{% code lang:javascript %}
function foo(){
    var hehe = 1;
    var lala = 2;
    bar = 2;
    return hehe + lala;
}

foo();
{% endcode %}

以上面这个函数为例，执行`foo`函数时，函数作用域链前端首先是该函数的变量对象，里面包含了局部变量`hehe`、`lala`以及对应的值，而`bar`为什么不在呢？因为它是未定义的，所以它直接被添加到全局环境的变量对象里面，也就是`foo`函数作用域链的第二个对象里面。
PS: 可见未定义变量会污染全局环境，所以声明变量不要漏掉`var`

###  作用域链的作用

作用域链的目的是保证当前执行环境对可访问的变量和函数的有序访问。
具体实现就是：函数内部标识符解析时会沿着作用域链一级一级地搜索。搜索过程始终从作用域链前端开始，直到最后一个对象即全局变量对象为止，如果找不到的话，就是undefined。
这样就可以保证内部变量的优先级始终大于外部变量

### 变量提升问题

{% code lang:javascript %}
var name = '123';
function foo(){
    console.log(name);
    var name = '456';
    console.log(name);
}

foo(); // undefined
       // 456
{% endcode %}

以上面这个函数为例，按常理理解，应该是先输出全局变量123，再输出局部变量456，然而第一个却输出了undefined，这是为什么呢？
因为函数在执行时会先根据局部变量创建作用域链，**这是在整个函数执行之前就完成的**，所以第一次输出时，该函数的变量对象中已经包含了name这个变量，从而导致解析时在作用域链第一个对象就停止，而不会访问到全局变量，同时那个时候局部变量name并没有赋值，所以会输出undefined。这种现象叫做变量提升。
PS: 在ES6中通过使用let标识符可以防止这个现象的出现

{% code lang:javascript %}
function foo(){
    name = 'yoo';
    console.log(window.name);
    var name = '456';
    console.log(name);
}

foo(); // undefined
       // 456
{% endcode %}

再看这个例子，这里面第一次赋值name没有定义，理论上应该是在window对象中。**但是因为下面的定义**，所以在函数创建时，那么这个变量已经在函数的作用域中，第一次赋值会成功赋值给函数作用域的name变量而不是window作用域中的name，所以调用window.name时会显示undefined。

## 作用域链的延长

部分语句可以在作用域链的前端临时增加一个变量对象，该变量对象会在代码执行后移除。主要由两种情况

* with语句
* try-catch中catch块

### with语句

> 平时有优化需要可以把所需对象存储在局部变量中，**不推荐使用with语句**，可能造成bug和性能损失。
> 详情可参考[with - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/with)

{% code lang:javascript %}
function foo(){
    var hehe = '?name=1';

    with(location){
        var url = href + hehe;
    }

    return url;
}

foo(); // www.some.com?name=1;
{% endcode %}

看上面这个例子，`with`语句吧`location`对象添加到了作用域链前端，因此在访问`href`时，其实是访问了`location.href`。在`with`语句执行完毕后，作用域链就返回之前的状态

### try-catch中catch块

{% code lang:javascript %}
try{
    someThing();
}catch(e){
    console.log(e);
}
{% endcode %}

当`try`代码块中的语句发生错误时，执行过程跳转到`catch`语句块，并且把一个异常对象添加到作用域链的头部。在`catch`语句执行完毕后，作用域链就返回之前的状态。

PS: 可以选择把错误处理委托给一个函数。这样的话只执行一条语句，并且没有访问局部变量，对性能的影响就比较小。

{% code lang:javascript %}
try{
    someThing();
}catch(e){
    handleError(e);
}
{% endcode %}

## 参考

* [JavaScript 开发进阶：理解 JavaScript 作用域和作用域链](http://www.cnblogs.com/lhb25/archive/2011/09/06/javascript-scope-chain.html)
* [with - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/with)
* [try...catch - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/try...catch)