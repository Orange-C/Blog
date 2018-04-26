---
title: js单元测试调研
date: 2016-06-10 13:30:36
categories: JavaScript
tags:
- JavaScript
- test
---

## 前言

之前我负责给项目搭建单元测试环境，对当前的web前端单元测试方案调查了一番，顺便看了一点E2E测试的方案。当时在团队内做了一次技术分享并且与java后端开发人员交流了一下关于测试的看法。我打算在这篇文章总结一下这次工作的收获。

## 关于测试的基础知识

解释一些关于测试的基础概念

### 什么是测试

[wiki](https://zh.wikipedia.org/wiki/%E8%BD%AF%E4%BB%B6%E6%B5%8B%E8%AF%95)上这么描述：

>软件测试的经典定义是：在规定的条件下对程序进行操作，以发现程序错误，衡量软件质量，并对其是否能满足设计要求进行评估的过程。

通俗地说，测试就是看软件是否达到要求，然而这要求有许多种，有的来自程序本身复杂度，有的来自客户需求，所以也诞生了许多测试方法，包括黑盒白盒，单元测试，集成测试等等。在复杂的软件产品中，测试是保证软件质量重要的一环。

<!-- More -->

### 关于自动化测试

关于测试，以前大部分都是程序员手动去模仿用户使用，来看软件是否符合预期，这样最原始最符合真实环境，但是也有很多缺点：

* 程序内部的测试很麻烦，因为人工测试大部分属于E2E，只能看到用户层的效果，很难看出程序内部的问题
* 无法保持测试的一致性，在多次测试中人往往不经意漏掉一部分测试

自动化测试，顾名思义就通过一个测试程序在软件更新之后自动运行测试。现在的测试基本都是自动化测试，测试成本也基本就是维护测试程序，但测试程序的复杂度因为用户需求不同会有很大的差别。
web前端在最初复杂度不高的时候因为大部分依赖后端，所以人工测试和后端测试基本就能满足需求。近几年web应用复杂度大大增加，前端测试也被重视了起来，出现了很多用于测试的工具。

### 关于TDD&BDD

说到测试往往伴随着这两个词，分别介绍一下：

* TDD，测试驱动开发（Test-driven development），倡导先写测试程序，然后编码实现其功能。测试驱动开发是戴两顶帽子思考的开发方式：先戴上实现功能的帽子，在测试的辅助下，快速实现其功能；再戴上重构的帽子，在测试的保护下，通过去除冗余的代码，提高代码质量。测试驱动着整个开发过程：首先，驱动代码的设计和功能的实现；其次，驱动代码的再设计和重构。
* BDD，行为驱动开发（Behavior-driven development），诞生于敏捷软件开发，它鼓励软件项目中的开发者、QA和非技术人员或商业参与者之间的协作。通过用自然语言书写非程序员可读的测试用例扩展了测试驱动开发方法。这让开发者得以把精力集中在代码应该怎么写，而不是技术细节上，而且也最大程度的减少了将代码编写者的技术语言与商业客户、用户、利益相关者、项目管理者等的领域语言之间来回翻译的代价。

换个比较有趣的方式看一下TDD和BDD
* **TDD**
* 一个测试工程师走进一家酒吧，要了一杯啤酒
* 一个测试工程师走进一家酒吧，要了一杯咖啡
* 一个测试工程师走进一家酒吧，要了0.7杯啤酒
* 一个测试工程师走进一家酒吧，要了-1杯啤酒
* 一个测试工程师走进一家酒吧，要了2^32杯啤酒
* 一个测试工程师走进一家酒吧，要了一杯洗脚水
* 一个测试工程师走进一家酒吧，要了一杯蜥蜴
* 一个测试工程师走进一家酒吧，要了一份asdfQwer@24dg!&*(@
* 一个测试工程师走进一家酒吧，什么也没要
* 一个测试工程师走进一家酒吧，又走出去又从窗户进来又从后门出去从下水道钻进来
* 一个测试工程师走进一家酒吧，又走出去又进来又出去又进来又出去，最后在外面把老板打了一顿
* 一个测试工程师走进一
* 一个测试工程师走进一家酒吧，要了一杯烫烫烫的锟斤拷
* 一个测试工程师走进一家酒吧，要了NaN杯Null
* 1T测试工程师冲进一家酒吧，要了500T啤酒咖啡洗脚水野猫狼牙棒奶茶
* 1T测试工程师把酒吧拆了
* 一个测试工程师化装成老板走进一家酒吧，要了500杯啤酒并且不付钱
* 一万个测试工程师在酒吧门外呼啸而过
* ……
* **BDD**
* 一个测试工程师走进一家酒吧，要了一杯啤酒，应该得到一杯啤酒
* 一个测试工程师走进一家酒吧，要了一杯鸡尾酒，应该得到一杯鸡尾酒
* 一个测试工程师走进一家酒吧，啥都没要，那就不理他

简单地说，TDD是比较传统的测试方案，写测试，写程序，重构，完成开发。而敏捷开发中，按TDD那种方法写测试太慢了，客户要看到原型的时候你还在写测试，所以诞生了BDD，将TDD中必要的软件功能测试拿出来，在用户正常需求的预期下进行测试，不测试程序本身的健壮性，这样的话能在满足用户需求的情况下最快地进行测试驱动开发。

## 前端单元测试工具介绍

关于测试程序的整个流程大致如下：

* 构建测试需要的环境（浏览器/node）
* 引入一个单元测试框架
* 引入一个断言库
* 引入测试需要的工具
* 引入被测试模块和依赖模块
* 编写断言
* 运行测试程序

下面我会简单介绍一下测试要用到的工具

### 测试执行工具karma

AngularJS团队开发的测试执行工具（test runner），他本身不具备测试功能，主要用于管理所有测试工具，包括测试框架，测试环境，编译工具等等，在比较复杂的测试需求下非常好用。

### js单元测试框架

* `QUnit`——jQuery团队开发的测试工具，在jQ时代用于jQ的单元测试十分好用，但是和jQ一样都是比较老的技术了
* `jasmine/jest`——`jasmine`是当前比较流行的测试框架之一，而`jest`是facebook团队在`jasmine`基础上演变开发出来的框架。
* `mocha`——现在最流行的测试框架之一，可自由选择断言库

PS：调研时因为有`react`的测试需求，所以我主要尝试了`jest`和`mocha`，在没上karma的情况下使用发现`jest`速度相对`mocha`慢了很多，同时刚好找到了几个比较好的`mocha`的例子，所以后面基本都是使用`mocha`

### js断言库

断言库即测试用的一种类似自然语言的API，用于编写易懂的测试程序

* `chai`——支持TDD（assert）和BDD（expect/should）的断言风格
* `should.js/expect.js`——两个比较轻量的BDD断言库
* `assert`——这是一个`node core`模块，用于node的TDD开发

### mock工具

测试单元往往依赖于外部模块，然后依赖模块可能因为某些原因不能直接使用，就需要对其进行mock，伪造依赖模块。例如测试ajax要模拟返回404的情况，这就需要mock实现。目前常用的mock工具是`sinon.js`

### react测试工具

* `React TestUtils`——facebook官方提供的测试工具集，可渲染单个react组件，包含了事件模拟API和组件状态模拟API，详情可戳[这里](https://facebook.github.io/react/docs/test-utils.html)
* `enzyme`——因为facebook官网API比较繁琐，所以airbnb团队对它进行了封装开发出了`enzyme`，react官网也推荐使用这个库。它对外提供三种渲染组件的方式
    * `shallow`——浅渲染组件，不渲染子组件，只包含第一层DOM结构
    * `render`——将整个组件渲染成静态HTML字符串，包含全DOM结构（其实使用的是大家熟知的node模块`cherrio`）
    * `mount`——将整个组件渲染成真实DOM，与真实环境相同

这三种渲染方式速度是递减的，可以根据自己的测试需求选择想要的组件渲染方式

## 测试实践1——非浏览器环境单元测试

### 前言

其实这个方案是在一开始我不知道`karma`的情况下使用的，在遇到了一系列问题再咨询后才知道了`karma`这个工具，后面基本都是使用`karma`运行测试。但是这个方案完全能满足大部分node环境下js的单元测试，换句话说就是没用到浏览器API的js模块。

### 测试工具

* `mocha`
* `chai`
* `sinon.js`
* `enzyme`
* `babel`——编译es6风格的测试文件
* `mochawesome`——生成酷炫的html测试报告（这个我好喜欢但是karma没有TAT）

上面的工具都直接用npm安装即可，其中`babel`安装需要配置一下

```
npm install babel-core babel-preset-es2015 --save-dev
```

在项目目录下新建一个.babelrc配置文件，内容如下

```json
{
    "presets": [ "es2015" ]
}
```

### 单元测试样例

所有的测试程序最好在项目目录的test文件夹下面，这样mocha能自动识别

* 简单的模块测试
```js
import { expect } from 'chai'

describe('add(a, b) Test', () => {
    it('1 + 1 = 2', () => {
        expect(add(1, 1)).to.equal(2)
    })
    it('2 + 3 = 5', () => {
        expect(add(2, 3)).to.equal(5)
    })
})
```
* react组件测试
```js
import { expect } from 'chai'
import { render } from 'enzyme'
import App from 'src/app'

describe('Enzyme Render', function () {
    it('Todo item should not have todo-done class', function () {
        let app = render(<App/>)
        expect(app.find('.todo-done').length).to.equal(0)
    })
})
```

接下来只需要运行如下命令即可运行测试
```
mocha --compilers js:babel-core/register
```

如果要用`mochawesome`生成报告，只需加上reporter属性，报告会在目录下的一个新文件夹生成
```
mocha --compilers js:babel-core/register --reporter mochawesome
```
酷炫的mochawesome报告（实际还有动效噢~）
![mochawesome](/blog/images/mochawesome.png)

关于mocha和enzyme的使用我就不做赘述了，自己用的也不怎么样，详情可以参考阮一峰的这两篇文章
* [测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)
* [React 测试入门教程](http://www.ruanyifeng.com/blog/2016/02/react-testing-tutorial.html)

### 该方案的一些缺陷

* **单纯的es6引擎无法识别webpack引入的非js模块**
因为开发的时候用webpack构建，所以项目的js文件中有不少引入模块是非js的，包括css，json等等，但是mocha只能用babel引擎，所以编译时遇到`import 'index.css'`这种语句就会直接挂掉，如果要为了适应测试而更改原项目结构这肯定是不佳的。
* **测试环境依为node，不能测试用到浏览器API的js文件**
在web前端的开发过程中，必然会用到大量的浏览器API，然而node环境下并没有`window`变量，所以所有用到这个变量的js文件都会挂掉。

综上所述，最终放弃了该方案，在咨询过程中得知了karma，搭建了最终的测试环境

## 测试实践2——虚拟浏览器环境单元测试

**提醒: 这份karma配置在windows下调用sinon.js会有路径错误，暂时没找到解决方法**

### 前言

karma的优点在于快速重载测试，多浏览器测试和**webpack预处理**。karma的配置异常繁琐，我的karma配置基本来自于[React 测试驱动教程](http://www.jianshu.com/p/6c74c96148c9)，这篇文章翻译自一篇国外文章[Test Driven React Tutorial](http://spencerdixon.com/blog/test-driven-react-tutorial.html?utm_campaign=Front+End+Newsletter&utm_medium=email&utm_source=Front_End_Newsletter_2)。

### 测试工具

* `karma`——测试运行工具
* `phantomjs`——提供虚拟webkit环境，不会打开浏览器
* `mocha`
* `chai`
* `sinon.js`
* `enzyme`
* `webpack`——编译测试文件

### `karma.config.js`解释

因为karma的具体配置过程在[React 测试驱动教程](http://www.jianshu.com/p/6c74c96148c9)已经讲得很详细了，所以我就大致解释一下karma.config.js的各项，方便更改配置

```js
// ./karma.config.js

var argv = require('yargs').argv;
var path = require('path');

module.exports = function(config) {
    config.set({
        // 浏览器环境设置，可以选择多个浏览器，如chrome，
        // firefox，ie等等，首先要下载karma插件并在下面的plugins里面声明才可用
        browsers: ['PhantomJS'],

        // 如果不添加--watch参数就只运行一次测试
        singleRun: !argv.watch,

        // 测试框架设置，同理也要先下载插件并声明
        frameworks: ['mocha', 'chai'],

        // 测试报告格式测试，同理也要先下载插件并声明
        reporters: ['spec'],

        // 加入babel和phantomjs的polyfill
        files: [
            'node_modules/babel-polyfill/dist/polyfill.js',
            './node_modules/phantomjs-polyfill/bind-polyfill.js',
            './test/*.js' // 声明watch测试文件的路径
        ],
        preprocessors: {
            // 希望用webpack预处理的文件
            // 测试时使用sourcemap方面debug
            ['./test/*.js']: ['webpack', 'sourcemap']
        },
        // webpack配置，直接使用你开发环境下的webpack配置即可
        // 或者重写一份更合适的，这样可以保证测试与开发的一致性
        webpack: {
             devtool: 'inline-source-map',
             resolve: {
                root: __dirname,

                extensions: ['', '.js', '.jsx'],

                // 为了让enzyme正常工作（不知道为什么总之加上就是了= =）
                alias: {
                    'sinon': 'sinon/pkg/sinon'
                }
            },
            module: {
                // 不要用babel编译sinon
                noParse: [
                    /node_modules\/sinon\//
                ],
                // 编译测试文件
                loaders: [
                    { test: /\.js$/, exclude: /node_modules/, loader: 'babel' },
                    { test: /\.css$/, loaders: ['style', 'css',    'postcss']},
                    { test: /\.json$/, loader: 'json'}
                ],
            },
            // 为了让enzyme正常工作（不知道为什么总之加上就是了= =）
            externals: {
                'jsdom': 'window',
                'cheerio': 'window',
                'react/lib/ExecutionEnvironment': true,
                'react/lib/ReactContext': 'window'
            },
        },
        webpackMiddleware: {
            noInfo: true
        },
        // 声明所有插件
        plugins: [
            'karma-mocha',
            'karma-chai',
            'karma-webpack',
            'karma-phantomjs-launcher',
            'karma-spec-reporter',
            'karma-sourcemap-loader'
        ]
    });
};
```

最后只要在`package.json`中加上这个命令即可
```json
"scripts": {
    "test": "node_modules/.bin/karma start karma.config.js --watch",
},
```
以后只需运行以下命令即可启动测试，并且能够在文件更新时自动进行测试
```
npm run test
```
启动测试：
![test](/blog/images/test.png)
文件更新时自动进行测试：
![testChange](/blog/images/testchange.png)
大功告成，至此已经成功搭建了测试环境

## 前端测试的一些难点

* **UI行为难以测试**
因为我目前的项目是一个基于地图的应用产品，经常遇到在地图上画点画线等需求，然而这种行为在测试用例里面很难表达出来，更不用说画图工具封装在地图API之内难以找到测试所需的DOM结构
* **DOM事件需要落实到具体的DOM节点**
众所周知，web前端测试肯定需要测试大量click事件change事件等等，然而在编写测试用例的时候这些事件肯定要声明触发的节点。然而DOM结构是非常多变的，一般来说DOM真正稳定的时候已经是开发末期了，所以这时候测试用例的维护变得非常麻烦，常常需要更要DOM节点名称

还有其他更多靠近UI层的测试可能还是需要人工或者其他工具来实现，同时不少测试工作还需要衡量维护测试文件的工作量。前端的测试仍然在发展中，未来或许会有更好的测试方案，目前取自己所需即可。

## 参考

* [软件测试 - 维基百科](https://zh.wikipedia.org/wiki/%E8%BD%AF%E4%BB%B6%E6%B5%8B%E8%AF%95)
* [测试驱动开发 - 维基百科](https://zh.wikipedia.org/wiki/%E6%B5%8B%E8%AF%95%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91)
* [行为驱动开发 - 维基百科](https://zh.wikipedia.org/wiki/%E8%A1%8C%E4%B8%BA%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91)
* [React TestUtils](https://facebook.github.io/react/docs/test-utils.html)
* [测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)
* [React 测试入门教程](http://www.ruanyifeng.com/blog/2016/02/react-testing-tutorial.html)
* [React 测试驱动教程](http://www.jianshu.com/p/6c74c96148c9)
* [Test Driven React Tutorial](http://spencerdixon.com/blog/test-driven-react-tutorial.html?utm_campaign=Front+End+Newsletter&utm_medium=email&utm_source=Front_End_Newsletter_2)