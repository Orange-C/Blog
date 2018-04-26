---
title: 谈谈react组件设计
date: 2016-10-02 14:55:06
categories: JavaScript
tags:
- JavaScript
- React
---

## 前言

从事react开发已经四个月有余，期间设计了不少业务组件，也魔改了一些别人的组件。在度过这一段非常react的开发过程之后，我想大致分享一下react组件设计的一些基础知识（以及坑）。

## 组件划分

在设计组件之前，我们首先要清楚在整个设计中哪些部分应该独立出来成为一个组件，哪些部分应该交给props控制，哪些部分是静态属性无需变动，我将其称之为组件划分。先看一下下面这个组件划分示意图↓

![component](/blog/images/component.png)

可以看到它将设计图划分为一个个嵌套的容器，其中有交互的部分如搜索框要成为一个组件，固定的属性比如表头的Name和Price是不变动的所以没有划分为组件，而是作为表格组件的静态样式。

## 组件分类

在划分之后我们脑海中对应会有这么一个组件树的样子出来。

```js
(
    <ProductTable>
        <Search />
        <Table>
            <Block>
                <BlockItem />
                <BlockItem />
            </Block>
            <Block>
                <BlockItem />
                <BlockItem />
            </Block>
        </Table>
    </ProductTable>
)
```
<!-- more -->

这就是组件最初始的状态，我写成了一个嵌套的样式，但我们真实使用的时候可能有两种用法：

* 单元组件，只用props控制，如上面的<Search />，它的内部可能有input和checkbox等结构，但是实际使用的使用并不需要知道内部结构，直接用props传递文档中定义的属性即可
* 容器组件，用嵌套结构，直接控制内部应该有哪些子组件，如上面的<Block />，<Table />以及最外层的<ProductTable />，都可以写成容器的形式。


两者是最基础的组件使用方法，理论上可以全写成第一种，或者全写成第二种。但是两者各有优劣，实际开发中我们需要判断哪种方式最适合当前的组件。

### 单元组件

我们常见的基础组件一般都是单元组件，通常可以独立使用。使用方法通常如下，组件的所有特性都通过props操控，组件的内部结构在文件中是固定的，常见用法如下
```js
(
    <Button size="small" type="primary" onClick={this.handle}>button</Button>
)
```

### 容器组件

顾名思义，容器组件通常作为容器配合单元组件使用，意味着单独的容器组件是无法使用的。常见用法如下

```js
(
    <Wrapper onChange={this.onChange} {...this.props}>
        <Square type="small"/>
        <Triangle />
        <Line />
    </Wrapper>
)
```

在使用<Wrapper />时我们定义了内部有<Square />、<Triangle />、<Line />这三个子组件，所以他们会包含在<Wrapper />组件内部的this.props.children里面，作为容器组件，在文件中它并不知道自己内部结构是怎样，它只负责处理内部state和外部props，并在有需要的时候把它们向下传递（如redux的actions）。我们也可以在声明的时候直接给子组件传递props，但是要注意和容器组件传递的props的覆盖问题。

PS:在容器组件中需要使用React.cloneElement手动把props赋给children，如下

```js
const childrenWithProps = React.Children.map(this.props.children,
    (child) => React.cloneElement(child, childrenProps)
)
```

### 优劣比较

* 单元组件使用方便，所有特性都通过props控制，适用于内部结构比较固定的组件。如按钮，表单元素，固定的数据展示页面等等。可参考antd的大部分基础组件。
* 容器组件使用相对繁琐，每次都需要定义内部结构，但是结构更加灵活，可以通过搭配实现多种组合，适用于内部可能有多种结构的大组件。如不定项的数据图表，结构多变的表单等等。可参考antd的Form组件或recharts的组件设计。

## 基于Decorator的扩展

### 容器组件开发的矛盾之处

在业务复杂，肯定会大量接触到容器组件配合单元组件的开发。但是这种开发其中还有一些矛盾之处：首先，作为整个组件来看，不少属性肯定是放在容器的state或者外部props中，这样能够传递到各个下层，这就意味着对于大部分逻辑函数肯定是要写在容器组件内部。但是同时我们的子组件是自定义的，如果要让某个子组件能够配合容器工作，肯定要在容器内部加上子组件的逻辑函数。**在子组件自定义的同时，容器组件内的逻辑和属性却越来越多，这就与之前的自定义相矛盾。**

### 将业务逻辑放在Decorator中

这个思想主要来自于这篇文章[基于Decorator的组件扩展实践](https://zhuanlan.zhihu.com/p/22054582?refer=purerender)，里面提出了一种组件组合式开发思想。大致思路是先编写单元组件，容器组件以及业务逻辑的Decorator，然后在使用时挑选三者所需组成一个特定的业务组件。可参考下图↓

![decorator-component](/blog/images/decorator-component.jpg)

Decorator是ES6新特性之一，能够接受一个class生成一个新的class，从而实现用Decorator返回一个继承了相应业务逻辑的class，具体实现如下
```js
const SearchDecorator = Wrapper => {
    class WrapperComponent extends Component {
        handleSearch(keyword) {
            this.setState({
                data: this.props.data,
                keyword,
        });
        this.props.onSearch(keyword);
        }

        render() {
            const { data, keyword } = this.state;
            return (
                <Wrapper
                    {...this.props}
                    data={data}
                    keyword={keyword}
                    onSearch={this.handleSearch.bind(this)}
                />
            );
        }
    }
  
    return WrapperComponent;
}

@SearchDecorator
class Search extends Component {
    render() {
        return (
            <Selector
                {...this.props}
            >
                <SearchInput />
                <List />
            </Selector>
        );
    }
}
```
在需要多个Decorator一起使用的时候，可以通过compose方法实现，如下
```js
const FinalSelector = compose(AsyncSelectDecorator, SearchDecorator, SelectedItemDecorator)(Selector);

class SearchSelect extends Component {
    render() {
        return (
            <FinalSelector
                {...this.props}
            >
                <SelectInput />
                <SearchInput />
                <List />
            </FinalSelector>
        );
    }
}
```

这样我们就能通过Decorator，容器组件，单元组件的自由配合，实现多种业务组件的实现。但是这样开发需要同事间的沟通配合以及详细文档的制定，所以在业务不是特别复杂或者开发进度比较赶的时候，还是推荐用前两种基础的方式进行组件开发。

## 参考

* [基于Decorator的组件扩展实践](https://zhuanlan.zhihu.com/p/22054582)
* [React Mixin 的前世今生](https://zhuanlan.zhihu.com/p/20361937)
* [ReactEurope 2016 小记 - 上](https://zhuanlan.zhihu.com/p/21379350)
