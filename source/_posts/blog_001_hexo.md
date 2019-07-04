---
title: Hexo静态博客方案
date: 2019-02-23 18:18:02
tags: hexo, github
categories: tools
---

## hello world

打开两年没更新的博客，简直不堪入目，索性推倒重来，先从如何搭静态博客开始。

## Hexo + Github Pages

Github提供了快速部署静态网页方法，[Github Pages](https://pages.github.com/)需创建一个Github Pages repository，与普通的repository唯一的区别是名字必须是 username.github.io。 [Hexo](https://hexo.io/)完美支持Markdown，渲染速度极快，依赖[Node.js](https://nodejs.org/en/)，可以一键部署，使用npm可以方便的管理插件。 但是静态博客存在一个问题，发布的内容是在本地渲染好的网页，如果更换电脑或者常用的电脑挂了就不能更新。我的解决方案是建一个新的repository，把本地hexo的整个working directory全部push到新的repository上，更新blog的操作：

```flow
node1=>operation: write blog
node2=>operation: hexo generate
node3=>operation: push to hexo repository
node4=>operation: hexo deploy

node1->node2(right)->node3(right)->node4
```

在一台新电脑上，安装hexo后，更新blog操作：

```flow
node1=>operation: write blog
node2=>operation: hexo generate
node3=>operation: push to hexo repository
node4=>operation: hexo deploy
node0=>operation: clone hexo repository

node0->node1->node2(right)->node3(right)->node4
```
和新建一个hexo项目执行hexo init不同的是，在clone hexo的目录下，执行
```
npm install
```

## Config

技术文档经常要写数学公式，Hexo默认采用”hexo-renderer-marked”插件渲染[Mathjax](https://www.mathjax.org/)公式，一个常见问题是公式内换行符会被错误解析，解决方法是修改node_modules/marked/lib/marked.js中464行，去掉对双反斜杠的转义：

```
escape: /^\\([\\`*{}\[\]()#+\-.!_>])/,
```

改为

```
escape: /^\\([`*{}\[\]()#+\-.!_>])/,
```

使用Mathjax公式需在Hexo的_config.yml中添加`mathjax: true`，同时在每个用公式的markdown文件的front-matter添加`mathjax: true`。

为了与[Typora](https://www.typora.io/)的markdown公式、流程图匹配，流程图采用插件[hexo-filter-flowchart](https://github.com/bubkoo/hexo-filter-flowchart) ，与Typora中的流程图语法基本一致。
Theme选了一个极精简[Maupassant](https://github.com/tufu9441/maupassant-hexo)，依赖`hexo-renderer-pug`和`hexo-renderer-sass`两个插件。Maupassant这个单词的意思竟然是莫泊桑……
