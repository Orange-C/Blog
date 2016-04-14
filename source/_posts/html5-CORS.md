---
title: 谈谈CORS跨域
date: 2016-03-22 14:04:14
categories: HTML&CSS
tags:
- CORS
- XMLHttpRequest
---

## 前言

前几天面试的时候被问到了关于跨域请求的问题，我就说用JSONP，iframe这些。面试官姐姐又追问我还有其他的吗，我表示不知道（一脸懵逼），后来告诉还有CORS，这几天就回来研究了下这个，发现是属于XMLHttpRequest2中的一个特性，而且果不其然老IE（IE8/9）是用其他API实现的。

## 什么是CORS

CORS全称是Cross-Origin Resource Sharing(跨域资源共享)，是W3C提出的跨域请求方案，需要服务器端的支持。与JSONP相比，主要有以下几点不同。
* JSONP只能实现GET请求。CORS支持所有类型的HTTP请求
* 使用CORS，开发者可以用普通的XMLHttpRequest发起请求和获得数据，相比JSONP有更好的错误处理
* JSONP兼容老的浏览器。不过考虑到IE8/9能够通过XDomainRequest实现CORS请求，所以主流浏览器基本实现CORS。

<!-- More -->

可以看出CORS相比于JSONP基本全是优点，以后的跨域请求应该都是用CORS实现。

## 创建CORS请求对象

因为CORS是属于XMLHttpRequest2的一部分，所以主要要做的就是区分出IE8/9和不支持XMLHttpRequest2的浏览器。

{% code lang:javascript %}
function createCORSRequest(method, url) {
    var xhr = new XMLHttpRequest();    
    if ('withCredentials' in xhr) {    
        // 支持CORS    
        // 检查XMLHttpRequest对象是否有“withCredentials”属性,withCredentials仅存在于XMLHTTPRequest2对象里    
        xhr.open(method, url, true);    
    } else if (window.XDomainRequest) {
        // XDomainRequest仅存在于IE中，是IE用于支持CORS请求的方式    
        xhr = new XDomainRequest();    
        xhr.open(method, url);    
    } else {    
        // 不支持CORS    
        xhr = null;    
    }    
    return xhr;    
}    

var xhr = createCORSRequest('GET', url);
if (!xhr) {    
    throw new Error('CORS not supported');
}
xhr.send(); // 发送请求
{% endcode %}

## 服务器支持

服务器端对CORS的支持主要通过设置HTTP头Access-Control-Allow-Origin实现，如果浏览器检测到相应设置，就能允许AJAX跨域请求。
HTTP头的设置方法可以参考这个网站[enable cross-origin resource sharing](http://enable-cors.org/)

在设置时最好限制请求的来源(如下)，这样就可以防止恶意站点对服务器通过XSS攻击

{% code %}
Access-Control-Allow-Origin: http://www.some.com
{% endcode %}

## 参考
* [HTML5安全：CORS（跨域资源共享）简介](http://www.cnblogs.com/yuzhongwusan/p/3677955.html)
* [XMLHttpRequest Level 2 使用指南 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)
