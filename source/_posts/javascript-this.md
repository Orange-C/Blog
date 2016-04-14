---
title: 谈谈JavaScript中的this
date: 2016-03-16 18:25:42
categories: JavaScript
tags:
- JavaScript
- this
---

## 何为this

一般而言，在JavaScript中，this指向函数**执行**时的当前对象。换句话说，这个关键字与函数的执行环境有关，与声明环境无关。所以this的指向要看如何去调用这个函数而不是声明。

## 不同调用方式中的this

### 作为对象的方法调用

把函数赋值给对象的一个属性，然后**通过该对象**调用该方法，此时函数的执行环境就是这个对象，所以this指向该对象

<!-- More -->

{% code lang:javascript %}
var name = 'lala';
var obj = {
    name: 'hehe',
    show: function(){
        console.log(this.name);
    }
}

obj.show(); // hehe
{% endcode %}

换个更清晰的写法，我们把声明和调用放在两个对象里面

{% code lang:javascript %}
var obj = {
    name: 'hehe',
    show: function(){
        console.log(this.name);
    }
}

var t_obj = {
    name: 'lala',
    show: obj.show
}

t_obj.show(); // lala
{% endcode %}

可以看到show虽然是在obj中声明的，但是通过t_obj调用了这个方法，所以此时this指向t_obj。


### 作为函数调用

我们将上面的代码改一下，将obj.show赋值给全局变量show再调用他，此时this绑定到全局对象

{% code lang:javascript %}
var name = 'lala';
var obj = {
    name: 'hehe',
    show: function(){
        console.log(this.name);
    }
}
var show = obj.show;
show();// lala
{% endcode %}

### 在函数内部的函数调用

在函数内部调用一个函数，比如在一个对象的方法里面调用一个函数时，this会指向全局对象（讲道理应该指向对象）。这是JavaScript设计比较坑的一个地方，平时经常使用命名一个新变量that替代this。

{% code lang:javascript %}
var name = 'lala';
var obj = {
    name: 'hehe',
    show: function(){
        var test = function(){
            console.log(this.name);
        }

        test();
    }
}
obj.show(); // lala
{% endcode %}

修正版：

{% code lang:javascript %}
var name = 'lala';
var obj = {
    name: 'hehe',
    show: function(){
        var that = this;
        var test = function(){
            console.log(that.name);
        }

        test();
    }
}
obj.show(); // hehe
{% endcode %}

### 作为构造函数调用

我们常使用new 构造函数名()来创建一个对象，此时函数中的this指向新创建的对象。如果不使用new，则和普通函数一样绑定到全局对象

{% code lang:javascript %}
function Foo(){
    console.log(this);
}
var test = new Foo();// test
Foo();// window
{% endcode %}

### 在`setTimeout`、`setInterval`和匿名函数中

在`setTimeout`,`setInterval`和匿名函数执行时的对象为全局对象，所以this也指向全局对象。

{% code lang:javascript %}
var name = 'lala';
var obj = {
    name: 'hehe',
    show: function(){
        setTimeout(function(){
            console.log(this.name);
        },500);
    }
}
obj.show(); // lala
{% endcode %}

### 函数调用`call`和`apply`方法时

两者的本质的就是改变函数当前的上下文环境即this，两者的区别是`call`接受一个个参数，而`apply`接受一个参数数组。

PS：使用`call`和`apply`函数的时候要注意，如果传递的 this 值不是一个对象，JavaScript 将会尝试使用内部`ToObject`操作将其转换为对象。因此，如果传递的值是一个原始值比如 7 或 'foo' ，那么就会使用相关构造函数将它转换为对象，所以原始值 7 通过`new Number(7)`被转换为对象，而字符串'foo'使用`new String('foo')`转化为对象

### 函数调用`bind`方法时

函数调用bind方法时会创建一个有相同函数体和作用于的函数，但是新函数的this**永久**指向bind的第一个参数，即使作为对象方法调用。

## 参考

* [Javascript中this关键字详解](http://www.cnblogs.com/justany/archive/2012/11/01/the_keyword_this_in_javascript.html)
* [深入浅出 JavaScript 中的 this](http://www.ibm.com/developerworks/cn/web/1207_wangqf_jsthis/index.html#ibm-pcon)
* [this - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)
