---
title: 实现element ui中el-tree组件在lazy模式下的刷新
description: element ui是一个PC端高质量的vue前端组件库，我个人非常喜欢。最近在使用tree组件时碰到一些麻烦——没有直接的API可以方便地在lazy模式下实现整树的节点重新加载。我们来看看怎么样实现这样的需求。
categories:
 - vue
tags:
 - vue
 - 前端
 - element
 - tree
---
# 前言
最近我在研究把我们公司内部的产品重构，需要做一个树型组件，类似像下面这样。
<div align="center">
	<img src="{{site.baseurl}}/assets/images/2019/09/1568613408954.jpg" alt="树组件效果图">
</div>
功能需求是：当在输入框内输入关键字搜索时，从服务端获取匹配的树节点数据，重新加载到树型组件当中。

# 已知的可用于刷新树节点的办法
tree组件提供了几种方式来加载树节点：
- 提供`data`属性，tree组件会按照`data`的数据结构渲染树节点
- 使用`updateKeyChildren`方法
- 使用`append`方法
- 提供`lazy`属性和`load`属性，当树组件`lazy`属性为`true`时，会使用`load`指定的方法渲染树节点

以上方式，都有可能用来实现我们开篇提出的需求。但是前三种方法基本被pass,因为前三种方法都是偏向本地数据渲染，而我们的数据都是lazy加载的。
> lazy加载（懒加载）的意思是:每次只加载当前层级或者当前节点的子节点的数据，更深层级的数据需要主动触发后再加载，比如：点击展开图标等，这样能提升性能。

# 本地数据渲染有什么问题
我们在做一个树组件时，不得不考虑的一个问题是：`树展开图标`，也就是开篇图中每个item前面的三角形符号,点击该符号会加载当前节点的子节点。

tree组件提供了一个属性`isLeaf`来代表当前是否叶子节点，但只在lazy模式下生效。也就是说，如果我们通过`data`属性渲染树节点，就无法通过`isLeaf`属性指定叶子节点，tree组件会自动根据数据的`children`属性的值来决定是否叶子节点。

在当前场景中，我们的数据都是远程加载，由于懒加载机制，我们无法提前获得子节点数据，`树展开图标`难以加载，所以暂时不考虑通过`data`属性来处理我们的需求。

# 通过懒加载方式处理
如我们之前第四点办法所说：
> 提供`lazy`属性和`load`属性，当树组件`lazy`属性为`true`时，会使用`load`指定的方法渲染树节点

`load`属性的值是一个函数，该函数会在首次加载树以及手动触发加载子节点时调用。该函数接收两个参数:
- `node`:是当前需要加载子节点的节点对象，如果没有提供有效的节点对象，则表示根节点
- `resolve`:是一个函数，该函数接收一个数组参数，该函数负责把参数中的数据渲染到`node`节点之下

这个函数提供给树组件之后，会由树组件自动调用。

有的同学可能已经看出，load函数可能就是契机。

那么问题来了：

# 如何手动调用load函数
`load`函数其实很好定义，代码如下：
```typescript
fLoadKeywords(node: Object, resolve: Function, search_str?: string) {
    let params = {};
    if (node.id) {
      params.id = node.id;
    }
    if (search_str) {
      params.name = search_str;
    }
    this.fKeywordsAction(params).then(data => {
      const trans_data = transforms.transformKeywords(data.content, node.id);
      resolve(trans_data);
    });
}
```

而我们监听搜索框输入时，手动调用`load`函数。监听搜索输入函数如下：
```typescript
fFilterKeywords(val: string) {
    // todo 清除根节点
    
    // 主动调用load函数，但是这里有一个问题，如何提供resolve参数？？？
    this.fLoadKeywords([], resolve, val);
}
```

在这一步，我们遇到了两个问题：
1. 我们需要根节点数据以便清除根节点
2. 我们需要知道`resolve`函数

那我们如何得到resolve函数？聪明的同学可能已经想到了，我们在load函数第一次被tree组件调用时，将resolve和根节点数据暂存起来。我们将`load`函数修改如下：
```typescript
fLoadKeywords(node: Object, resolve: Function, search_str?: string) {
    // 将resolve函数存储在别的地方，在搜索的时候会使用
    if (!this.tree.resolve) {
      this.tree.resolve = resolve;
    }
    let params = {};
    if (node.data && node.data.id) {
      params.id = node.data.id;
    }
    if (search_str) {
      params.name = search_str;
    }
    this.fKeywordsAction(params).then(data => {
      // console.log(data);
      const trans_data = transforms.transformKeywords(data.content, node.id);
      
      // 将根节点数据暂存起来
      if (node.id === 0 || node.length === 0) {
        this.tree.data_level1 = trans_data;
      }
      resolve(trans_data);
    });
}
```

再将监听搜索框函数修改一下:
```typescript
fFilterKeywords(val: string) {
    // 清除节点
    const tree = this.$refs.tree_keywords.$refs.tree;
    if (this.tree.data_level1 && this.tree.data_level1.length) {
      this.tree.data_level1.forEach(d => {
        tree.remove(d);
      });
    }
    
    this.fLoadKeywords([], this.tree.resolve, val);
}
```

# 结束语
总体来说，比较奇技淫巧。

但我觉得问题的根源是，`树展开图标`和`lazy`模式之间的矛盾：如果在`非lazy`模式下允许通过`isLeaf`属性指定叶子节点，那这个问题就不再是问题，我们可以通过`非lazy模式`来解决当前的问题。

所以我觉得树组件更好的设计是：
> isLeaf: 指定节点是否为叶子节点，与`lazy`无关。在`lazy = false`的情况下，如果没有显式指定`isLeaf`属性， 则根据`children`属性的值判断是否为叶子节点。