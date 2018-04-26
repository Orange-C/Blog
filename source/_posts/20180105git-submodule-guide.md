---
title: git submodule 使用指南
date: 2018-01-05 16:58:12
categories: tools
tags:
- git
---

## 前言

`submodule`是git自带的子模块系统，我们先看官网对于`submodule`的[定义](https://git-scm.com/docs/gitsubmodules)。

> A submodule is a repository embedded inside another repository. The submodule has its own history; the repository it is embedded in is called a superproject.
> submodule是嵌入在另一个项目中的项目，它拥有自己的git历史信息。而被嵌入的项目叫做上层项目。

这一段比较拗口，我用个人的理解再描述一下：我们可以在一个git项目中嵌入另一个git项目，被嵌入的叫做`submodule`，因为`submodule`本来就是一个**独立可用的git项目（记住这一点）**，所以它有自己的历史信息（branch，commit，etc.），也可以进行基本的git操作（pull/push，merge，etc.）。唯一不同的就是被嵌入的上层项目会记录submodule的改动（以commit ID的形式），并把它作为自身改动提交上去。

<!-- more -->

说到这里大家可能会想到npm，maven等等依赖管理的工具，因为他们也会帮助项目记录其他项目（也就是常说的依赖）的信息并一起提交。所以submodule也可以看成一个比较弱但更自由的依赖管理，它没有限制位置，格式，版本信息，模块如何引入等等，只要将其加到项目中就作为项目的一部分代码一起开发。这个博客的主题Next就是用submodule引入的，可以看到submodule的代码可以放在项目的任意位置，如下图:

![0](/blog/images/20180105-0.png)

submodule使用起来非常自由，但是伴随自由而来的是非常多的坑，团队成员每个人必须对git以及submodule基本操作有足够的了解，才能将其应用到多人开发的生产环境中。下面补充一些基本知识再讲一下常用的操作。

## 基础知识

### `submodule`与`commit ID`

众所周知git项目里面，文件如果有改动，就会产生一个diff信息，可以用`git diff`和`git status`看到。但是submodule不同，它自身就有完整的历史信息，所以上层项目不需要也不会去跟踪它里面具体的改动，那么上层项目如何跟踪submodule的变化呢？答案就是**commit ID**。

git每生成一个commit，都会生成一个唯一的hash值，叫做commit ID，这也是git作为版本管理系统的基础，每一个commit ID相当于一个版本号。**当且仅当submodule生成了一个不同的commit**，上层项目中才会生成一个可以提交的diff信息，如下图：

![1](/blog/images/20180105-175303@2x.png)

可以看到diff信息中显示submodule的commit ID发生了变化，并显示了变化的commit信息。这就是submodule存储在在上层项目中的信息，后续的更新等操作都依赖这个commit ID。这时候就需要注意以下问题：

1. 上层项目commit之前**必须先commit submodule的改动**，否则submodule改动就没有记录在上层项目中
2. 最后push所有时，**submodule改动过的commit必须push到远程**，否则上层项目更新submodule时会提示在远程找不到对应的commit ID

### `submodule`与`branch`

与我们平时常用的开发方式不同，可以看到在submodule的开发过程中没有branch这个概念，上层项目记录的只有submodule当前的commit ID。不管submodule当前处于哪个branch，只要最新的commit ID相同，就认为是相同的。这也是大多数人第一次接触submodule开发时容易踩的坑。不过虽然没有branch这个概念，多人开发时还是建议使用各自的branch以便协作。

<i>ps：submodule可以配置成不记录commit ID而是跟踪固定branch，某些场景下更方便但这种方式其实是一种不稳定的依赖管理，不推荐使用，所以不写在正文中，相关介绍见附录1。</i>

## 常用操作

### 添加

```bash
git submodule add <repo-path> <submodule-path>

e.g.:
git submodule add git@github.com:Orange-C/hexo-theme-next.git themes/next
```

repo路径用ssh或者http形式都可以，初次添加后会在项目根目录生成`.gitmodules`文件，记录submodule的路径信息等，这个文件主要用于后面的`init`命令，具体内容如下：

```
[submodule "themes/next"]
    path = themes/next
    url = git@github.com:Orange-C/hexo-theme-next.git
```

### 初始化目录

```bash
git submodule init
```

根据项目的`.gitmodules`文件初始化各个submodule的目录（并不更新文件）。主要在本地没有相应submodule路径时使用（比如第一次clone、新增了submodule等等）

### 同步

```bash
git submodule update
```
根据上层项目存储的commit ID将submodule更新到相应commit ID的状态，通常在init之后使用，或者上层项目checkout分支之后使用。update理论上和pull类似，如果submodule本地有修改没保存就会失败。

**注意事项：**
1. **如果update操作产生了任何实质的更新**，则更新后submodule会处于`detached HEAD`状态，不track任何分支。所以此时如果要在submodule内开发，请手动checkout至相应分支。
2. 上层项目不同分支存的commit ID可能不同，**但是切换分支不会对子模块进行任何操作**，如果存的commit ID不同，在checkout之后会立即产生一个diff，此时就需要update同步。
3. 在进行add操作时，会自动进行init和update操作，此时submodule会默认处于master分支下。

### 开发

和普通的git项目一样，submodule的开发只需要进入相应的目录进行日常的git pull/push/add/commit操作即可。

在上层项目push前，必须保证submodule先push，git push有一个参数`recurse-submodules`，用这个参数就可以在上层项目中提交submodule的更改：
```bash
# 检查submodule是否已经提交，如果没有则提交失败
git push --recurse-submodules=check 
# 检查submodule是否已经提交，如果没有则先提交submodule再提交上层项目
git push --recurse-submodules=on-demand 
```

### 删除

因为git缓存等原因，submodule经常会出现没有彻底删除的情况，下面这几行命令可以在项目中彻底删除一个submodule。

```
<submodule-path> eg: theme/next
git submodule deinit -f <submodule-path> 
rm -rf .git/modules/<submodule-path>
git rm -f <submodule-path>
```

## 总结

submodule是一个非常自由和方便的依赖管理，适合个人开发和小型项目中使用。但是在团队人数较多，开发人员了解不足，项目依赖多且杂的情况下，还是推荐使用npm等成熟的依赖管理工具。

## 附录1：submodule跟踪branch开发

add时使用`-b`参数确定跟踪分支
```
git submodule add -b <branch> <repo> <project-path>
```
可以看到`.gitmodules`文件中多记录了一个分支信息如下
```
[submodule "themes/next"]
    path = themes/next
    url = git@github.com:Orange-C/hexo-theme-next.git
    branch = master
```
同步时需要加上`--remote`参数，从远程直接拉取branch最新commit ID的文件
```
git submodule update —remote
```
**再次声明，这种方式是一种不稳定的依赖管理，不推荐多人开发时使用。**

## 参考

* [Git - gitmodules Documentation](https://git-scm.com/docs/gitmodules)
