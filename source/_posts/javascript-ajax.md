---
title: JavaScript实现Ajax
date: 2016-03-15 20:44:28
categories: JavaScript
tags:
- JavaScript
- ajax
---

## 前言

原生javascript实现ajax是一个比较老套但经常出现的面试题，主要是为了筛选掉长期依赖类库编程的前端。对于新手也是个不错的练习。

## Ajax实现步骤

### 创建XMLHttpRequest对象

{% code lang:javascript %}
var XHR;
if(window.XMLHttpRequest){
    XHR = new XMLHttpRequest();
}else if(window.ActiveXObject){// IE6
    XHR = new ActiveXObject('Microsoft.XMLHTTP');
}
{% endcode %}

<!-- More -->

PS：ActiveXObject对象在IE11中已移除

### 发送请求

首先调用`open`方法，接受三个参数

* `type|String`：表示请求的类型，包括get，post等
* `url|String`：表示请求的URL
* `async|Boolean`：表示是否异步发送请求

然后调用`send`方法，接受一个参数即要作为请求主体发送的数据

{% code lang:javascript %}
if(type == 'GET'){
    //拼接GET方法的URL
    if(typeof(data) != 'undefined'){
        url += '?';
        for(i in data){
            url += i + '=' + data[i] + '&';
        }
        url = url.substring(0,url.length - 1);
    }
    XHR.open(type,url,true);
    XHR.send(null);
}else if(type == 'POST'){
    XHR.open(type,url,true);
    XHR.send(data);
}
{% endcode%}

PS: 理论上GET方法也可以有body，但一般来说约定GET的参数都放在URL上，所以type为GET时send的参数一般为null。

### 收到响应

当时XHR的`readyState`改变时就会触发`readystatechange`事件。通常`readyState`为4时表示已经接收到所有响应数据。可以根据XHR的`status`属性（即HTTP状态码）确定请求是否成功。

{% code lang:javascript %}
XHR.onreadystatechange = function(){
    if(XHR.readyState == 4){
        if(XHR.status >= 200 && XHR.status < 300 || XHR.status == 304){
            // 请求成功
        }else{
            // 请求失败
        }
    }
}
{% endcode %}

到这里整个AJAX已经完成了，接下来就是调用回调函数实现需求了。

## 一个小型的ajax函数实现

曾经写的一个小作业

{% code lang:javascript %}
function ajax(url, options) {
    var XHR,i;
    if(window.XMLHttpRequest){
        XHR = new XMLHttpRequest();
    }else if(window.ActiveXObject){
        XHR = new ActiveXObject('Microsoft.XMLHTTP');
    }

    if(typeof(options.type) === 'undefined'){
        options.type = 'GET';
    }

    if(options.type == 'GET'){
        if(options.data){
            url += '?';
            for(i in options.data){
                url += i + '=' + options.data[i] + '&';
            }
            url = url.substring(0,url.length - 1);
        }
        XHR.open(options.type,url,true);
        XHR.send(null);
    }else if(options.type == 'POST'){
        XHR.open(options.type,url,true);
        XHR.send(data);
    }

    XHR.onreadystatechange = function(){
        if(XHR.readyState == 4){
            if(XHR.status >= 200 && XHR.status < 300 || XHR.status == 304){
                if(options.onsuccess){
                    options.onsuccess(XHR.responseText,XHR);
                }
            }else{
                if(options.onfail){
                    options.onfail(XHR.responseText,XHR.status);
                }
            }
        }
    }
}

// 使用示例：
ajax(
    'someURL',
    {
        type: 'POST',
        data: {
            name: 1,
            test: 2
        },
        onsuccess: function (response, xhr) {
            console.log(response);
        },
        onfail: function(response, status){
            console.error('AJAX ERROR ' + status + ': ' + response);
        }
    }
);
{% endcode %}

## 总结

第一次写技术博客，边写边改代码，不得不说写的时候回头看代码真是漏洞百出……
PS：以后在博客里加上emoji，没表情简直玩不起来。
