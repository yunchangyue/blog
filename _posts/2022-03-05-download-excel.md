---
title: 实现react组件并发布到npm
description: 第一次做内部公共组件库，碰到了一些问题。在此记录一下。
categories:
 - react
tags:
 - react
 - 前端
 - 组件库
 - npm
---
## 前言
最近想要把一个下载Excel的组件逻辑提取到公共组件库中，以方便组内的其它小伙伴使用。这也是我第一次做这种公共组件并发布npm，期间碰到一些问题，在此记录。

## 组件库的功能需求
组件库的功能需求比较简单，就是可以将本地数据（数组）或者通过`ajax`请求得到的远程数据流（Blob）装载到Excel(`xlsx`)文件中进行下载下来。

## 核心功能实现
> 为避免长篇大论，这里不记录详细的实现细节, 只简单说实现的方向。

### 如何实现本地数据加载到excel?
在使用本地数据时，使用了`xlsx`这个库进行`excel`的数据写入及文件生成。

远程`Blob`数据因为已经是后端返回的excel格式的数据流，所以不需要使用该库进行处理，直接装载到文件下载即可。

### 如何实现下载文件?
通过html`<a>`标签,使用其`download`属性进行下载。大概实现流程是：

1. 通过`javascript`新建一个临时`a`标签
2. 指定`a`标签的`download`属性和`href`属性
3. 通过`javascript`触发`a`标签的`click`事件,从而触发下载
4. 移除临时`a`标签

## 组件打包
组件使用`webpack`打包，但是打包完成后，发现主文件比较大，大约900多k，原因是`react`和`xlsx`源代码被打包进了组件里，这好像有些不太合理。

## 组件优化的思考

### 如何减小代码体积？
`react`代码好像不应该被打包进组件中，因为我们的组件是给别人的应用使用的，别人的应用中肯定已经引入了`React`.解决办法是使用`webpack`中的`externals`配置进行处理:

```javascript
externals: {
    react: 'react'
}
```

`webpack`会认为`react`是一个被外部引用的依赖，从而不对其进行打包。

值得注意的是，建议在组件库的`package.json`中添加`peerDependencies`配置，以保证安装运行时不会出错。

```json
{
    "peerDependencies": {
        "react": ">=16.8.0"
    }
}
```

> 以上`peerDependencies`配置的通俗意思是：“如果你安装了我的组件库，则你最好也安装react>=16.8.0”。如果在安装该组件库时你的应用中没有安装`peerDependencies`中指定的依赖，在不同`npm`版本下的表现不太一致：在`npm@7`以下的版本会在控制台显示警告信息，在`npm@7`及以上则会默认安装该依赖。

### 组件代码依然较大，怎么破?
不打包`react`最终输出的代码依然有900k左右，这是因为`xlsx`写入文件功能非常全面导致代码量较大，但是`xlsx`的代码又无法进行再分割。

如何才能使组件更优化呢？

既然代码不能更加精减，那就从结构上入手吧：

我把这两个功能拆分成两个组件，分别实现本地数据下载和远程Blob数据下载，这样当用户只使用远程Blob数据时，就可以只引入这个远程Blob数据组件，这个组件没有导入`xlsx`,所以代码量较小，而本地数据下载组件则会被`webpack`打包时`tree-shaking`掉。当然，如果要使用本地数据下载组件，代码量依然是比较大的。😂

## 组件测试

为了达到更完整全面的测试目标，我通过模拟真实使用场景的方式进行测试：

1. 新建一个测试项目
2. 在组件库根目录中使用`npm link`将该组件库链接成一个本地全局包
3. 在测试项目中使用`npm link [包名]`，将组件库链接到测试项目中
4. 然后像正常项目的`import`使用即可

这样可以测试到安装、导入、功能、`tree-shaking`优化等全套流程。

## 组件发布

我将组件发布到了组内的npm平台。这个事情相对就比较容易了。

```shell
# 先进行登录
npm login

# 再进行发布
npm publish
```

如果对自己的代码没有信心，可以先发一个测试版。

```shell
npm publish --tag beta
```

## 结束
好了，结束了。有什么问题可以留言一起探讨。

明天就开始新的一周了，要努力加油。😀