---
title: 前端面试复习-2-HTML&CSS
date: 2016-03-27 13:34:07
tags:
---

## HTML(5)

### HTML5新特性

* 语义化的标签（header,nav,footer,aside,article,section）
* 拖放事件（drag类事件）
* 跨文档消息传递（postMessage）
* 媒体元素（audio，video标签）
* 画布（Canvas） API
* 地理位置定位（navigator.geolocation）API
* 历史状态管理（更新history对象，pushState等）

<!-- More -->

### HTML5离线存储技术

在用户离线时，可以正常访问站点，联网时可以更新缓存。使用方法就是在html标签中加上manifest属性,如下
{% code lang:html %}
<!DOCTYPE HTML>
<html manifest="cache.manifest">
...
</html>
{% endcode %}
cache.manifest文件格式如下
```
CACHE MANIFEST

CACHE:

js/app.js
css/style.css

NETWORK:
resourse/logo.png

FALLBACK:
/ /offline.html
```
离线存储的manifest一般由三个部分组成:
1.CACHE:表示需要离线存储的资源列表
2.NETWORK:表示在它下面列出来的资源只有在在线的情况下才能访问，他们不会被离线存储
3.FALLBACK:(示例里面'/ /'不是注释)表示如果访问第一个资源失败，那么就使用第二个资源来替换他，比如上面这个文件表示的就是如果访问根目录下任何一个资源失败了，那么就去访问offline.html。

### Cookie和Web Storage

Cookie，主要用途是保存登录信息，大小限制为4kb左右。一般由服务器生成，可设置失效时间，如果客户端生成的话，默认是浏览器关闭后失效。每次都会携带在HTTP头信息中，所以Cookie太大的话会导致http请求变得很大带来性能问题。

Web Storage包含sessionStorage和localStorage。
* sessionStorage 为每一个给定的源（given origin）维持一个独立的存储区域，该存储区域在页面会话期间可用（即只要浏览器处于打开状态，包括页面重新加载和恢复，浏览器或tab页关闭后，sessionStorage清空）
* localStorage 与session类似，但是在浏览器关闭，然后重新打开后数据仍然存在。

Web Storage包含以下方法：setItem(),getItem(),removeItem(),clear()

同时Web Storage提供了storage事件，在创建，更新，删除数据项时会触发该事件（重复设置相同的键值不会触发），clear方法至多触发一次该事件。

## DOM

这部分知识有些涉及到IE和其他浏览器不同的实现方式，有不少细节我还是不太了解，所以就大致复习一下常用的DOM操作

### 获取元素的方法

* getElementById() Document能调用
* getElementsByTagName() 注意有s，返回HTMLCollection Document和Element能调用
* getElementsByClassName() 注意有s 返回HTMLCollection Document和Element能调用
* querySelector() 只返回获取的到第一个元素
* querySelectorALL() 返回NodeList

querySelector和querySelectorALL使用主要通过CSS选择符,可以通过Document和Element类型的节点调用它们。

### 节点类型

介绍几种常用的节点类型

1. Document类型，表示整个文档，浏览器中document对象是HTMLDocument（继承自Document类型），同时也是window的一个属性，可以全局访问。document的一些常用属性:
    * title 表示文档标题
    * body 表示body标签
    * URL 表示页面的URL，不可修改
    * referrer 表示来源页面的URL，不可修改（HTTP协议中是Referer，其实是当年拼错了Orz）
    * domain 表示URL中的域名，可修改但是不能设置URL中不包含的域，而且不能把loose的域设置为tight的域
    * forms 文档中form标签的集合
    * images 文档中img标签的集合
    * links 文档中带href特性的a标签的集合

2. Element类型，元素类型，即常见的div,p标签等，常用属性包括id和className。元素通常包含特性，可以通过getAttribute(), setAttribute()和removeAttribute()操作（HTML5要求自定义特性应加上data前缀）。

3. Text类型，包含纯文本内容，包括换行符空格等。通常那些没有被标签包裹起来的部分就是文本节点

4. DocumentFragment类型，这个类型用来表示一个轻量文档，可以包含和控制节点。可以把它用作仓库，把需要添加的节点逐个放入其中，然后再将其添加到文档中，例如添加表格，这样可以避免浏览器反复渲染。

### 节点操作

NodeList对象是基于DOM结构的动态查询，是双向绑定的。

* document.createElement() 创建元素节点，参数例如"div"
* document.createTextNode() 创建文本节点，参数字符串，可包含html行内标签。
* childNodes NodeList对象，保存了一组有序子节点，包含文本节点
* children 与childNodes类似，但是不包括文本节点
* appendChild(newNode) 将newNode添加到childNodes的末尾
* insertBefore(newNode,someNode) 把newNode插入到somNode之前，如果somNode为null，则放在最后
* replaceChild(newNode,someNode) 把someNode替换成newNode，其中someNode会被移除
* removeChild(someNode) 移除某个节点
* innerHtml 表示标签内部的内容
* outerHtml 表示整个标签的内容，包含标签

### 如何优化DOM操作的性能

* 避免反复使用DOM查询操作，把结果用变量缓存
* 避免大量使用会造成重绘的DOM操作
* 尽量使用ID选择器

## 事件

### 关于事件冒泡和事件捕获？

* 事件冒泡指的是事件开始时由最具体的元素接受，然后逐级向上传播到不具体的元素。也可以说是在DOM树中从最底层的子节点一直“冒泡”到最顶层的父节点
* 事件捕获与时间冒泡相反，事件开始时由最顶层的节点接受一直向下传播到事件的实际目标

### 事件绑定的方法？

DOM0级绑定方法，即直接给元素相应的ontype类型赋值
{% code lang:javascript %}
somenode.onclick = function(){
    ...
}
{% endcode %}
DOM2级绑定方法
* addEventaddEventListener(type,listener,useCapture)，参数type表示为“click”这种类型，参数useCapture表示是否使用事件捕获，默认为false，可不加。
* attachEvent(type,listener)，**这种方法主要为了兼容IE8**，IE9可用第一种方法，其中参数type表示为“onclick”这种类型。
{% code lang:javascript %}
function addEvent(element, event, listener) {
    if(element.addEventListener) {
        element.addEventListener(event,listener,false);
    }else if(element.attachEvent){//IE8
        element.attachEvent('on' + event, listener);
    }else{
        element["on" + event] = listener;
    }
}
{% endcode %}

### 关于事件对象event？

浏览器会将一个事件对象event传给事件处理程序,event对象包含的常用属性和方法如下：
* currentTarget 表示当前处理事件的元素，即绑定的元素，时间处理程序中对象this始终等于这个值
* target 表示事件的目标，即事件的实际触发元素
* type 表示事件的类型
* cancelable 表示是否可以取消事件的默认行为
* preventDefault() 取消事件的默认行为

### 什么是事件委托？

事件委托就是把事件处理程序绑定在父元素上，通过事件冒泡来获取子元素的事件触发事件处理程序。通常用于列表项中。好处如下
* 提高性能，不用循环逐个绑定相同事件
* 新元素拥有原来绑定的事件，这样就不用每次都为添加的新元素绑定事件

{% code lang:javascript %}
var list = document.getElementsByTagName('ul')
//list不是数组是HTMlCollection，所以不能用Array的方法循环如foreach和map
list[0].addEventListener('click',function(event){
    event.target.style.backgroundColor = 'red';//点击li后将其北京改为红色。
    console.log(event.currentTarget) // ul
    console.log(this) // ul
    console.log(event.target) //li
})
{% endcode %}

## CSS(3)

### CSS选择器有哪些？

* 元素选择器 h1
* id选择器 #someid
* 类选择器 .someclass
* 属性选择器 [attr=value]，[attr^=value]以value开头，[attr$=value]以value结尾，[attr*=value]包含value子串，[attr|=value]值为value或者以value-开头，[attr~=value]值用空格分割其中一个为value
* 后代选择器 div p 表示div后代中的所有p
* 子元素选择器 div>p 表示div直属后代中的所有p
* 相邻元素选择器 div+p 表示跟在div后面的p

多个选择器之间用``,``分割表示多个选择器共享属性。多个选择器之间不分割表示合并为一个选择器。选择器优先级tag>id>class。

### CSS伪元素有哪些？

* :after 匹配该元素的一个虚拟的最后子元素，配合content属性使用，默认为行内元素
* :before 匹配该元素的一个虚拟的最先子元素，与:after类似
* :first-line 匹配元素的第一行
* :first-letter 匹配元素的第一个字符
* :selection 匹配用户鼠标选中的部分

### CSS伪类有哪些？

* :link 未被访问的链接
* :visited 已被访问的链接
* :active 被激活的元素，通常为鼠标按下至松开的那段时间
* :hover 用户将鼠标移至其上方时
* :focus 元素成为焦点时
* :first-child 元素为其父元素的第一个子元素时
* :nth-child(an+b) 匹配为其父元素第an+b个子元素的元素
* :nth-last-child(an+b) 同上但是顺序相反
* :first-of-type 匹配元素中所有子元素类型第一个出现的元素
* :last-of-type 匹配元素中所有子元素类型最后一个出现的元素


### position属性有哪些？

* static 默认值，元素处于正常的文档流之中，top, right, bottom, left 和 z-index 属性无效。
* relative 元素相对于原本的正常位置定位，不改变布局，这样会在此元素原本所在的位置留下空白，对display为表格型的元素无效
* absolute 不为元素预留空间，元素相对于与它最近的非static定位的祖先元素来定位。元素可以设置外边距（margins），并且不会与其他边距合并（即形成一个BFC）
* fixed 不为元素预留空间，相对屏幕视窗来定位，在屏幕滚动时位置不变
* sticky （新属性试验中，目前仅firefox实现）平时为relative，在特定条件变为fixed，类似于现在的浮动条

### box-sizing属性值有哪些？
* content-box 默认值，标准盒模型，width和height表示content的宽高，不包括padding，border，margin。
* border-box IE怪异模式（Quirks mode）使用的盒模型，width和height包括padding和border，不包括margin

### 如何在JavaScript中更改样式？

访问元素的style属性，这个对象是CSSStyleDeclaration，包含了通过style特性指定的所有样式，所有短划线的CSS属性名转换为驼峰形式，如（background-color在JavaScript中为backgroundColor）

### 谈谈flex？

又称弹性盒，可以自动调整子元素高宽，主要概念就是关于水平垂直两个轴的布局，属性值都比较方便实用，布局思路也很清晰。具体用法参考[MDN -  flex](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flexible_Box_Layout/Using_CSS_flexible_boxes)和[阮一峰前辈的教程](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html?utm_source=tuicool)

### 清除浮动的方法？

我一般在父元素加一个clearfix类，用after伪元素，用zoom触发hashLayout兼容IE6-7
{% code lang:css %}
.clearfix:after{
    content: ".";
    display: block;
    height: 0;
    clear: both;
    visibility: hidden;
}
.clearfix{
    zoom: 1;
}
{% endcode %}

### 写一个常见的三列布局？

用float实现（兼容比较好）：
{% code lang:html %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>test</title>
    <style type="text/css">
    .left{
        float: left;
        width: 100px;
        margin-left: -100%;
        background-color: blue;
    }
    .right{
        float: left;
        width: 100px;
        margin-left: -100px;
        background-color: red;
    }
    .wrapper{
        float: left;
        width: 100%;
    }
    .main{
        margin: 0 110px;
        background-color: #ccc;
    }
    </style>
</head>
<body>
    <div class="wrapper">
        <div class="main">main</div>
    </div>
    <div class="left">left</div>
    <div class="right">right</div>
</body>
</html>
{% endcode %}

用flex实现（方便）：
{% code lang:html %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>test</title>
    <style type="text/css">
    .wrapper{
        display: flex;
    }
    .main{
        flex: auto;
        background-color: #ccc;
    }
    .left{
        order: -1;
        flex: 0 1 100px;
        margin-right: 10px;
        background-color: blue;
    }
    .right{
        flex: 0 1 100px;
        margin-left: 10px;
        background-color: red;
    }
    </style>
</head>
<body>
    <div class="wrapper">
        <div class="main">main</div>
        <div class="left">left</div>
        <div class="right">right</div>
    </div>
</body>
</html>
{% endcode %}
