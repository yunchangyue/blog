---
title: css伪类 :focus-within
description: :focus-within伪类是什么?与:focus伪类比较接近。:focus是当元素获得焦点时应用样式，而:focus-within是当前元素或者其子元素获得焦点时应用样式。
categories:
 - css
tags:
 - css
 - 前端
 - focus
 - focus-within
---

## :focus-within伪类是什么
与`:focus`伪类比较接近。

`:focus`是当元素获得焦点时应用样式，而`:focus-within`是当前元素或者其子元素获得焦点时应用样式。

## :focus伪类
`:focus`伪类大家都比较熟悉，比较常见的场景是input输入框获得焦点时高亮显示:
```css
.input {
  border: 1px solid #ccc;
  height: 30px;
  -webkit-border-radius: 20px;
          border-radius: 20px;
  padding: 0 12px;
  -webkit-transition: border .3s;
  -o-transition: border .3s;
  transition: border .3s;
}
.input:focus {
  border-color: #409EFF;
}
```
效果如图所示：
![input高亮]({{site.baseurl}}/assets/images/2018/08/focus.gif)

效果看起来不错。

## 问题进阶
现假设有如下需求,你会怎么做?
![复杂场景的input高亮]({{site.baseurl}}/assets/images/2018/08/focus-within.gif)

在本题中，圆角父容器中包含三个元素: 搜索图标，输入框和下拉展开按钮。需要达到的目标是，在输入框获得焦点时，父容器边框颜色改变。

**我相信你马上就能说出答案——**通过`javascript`监听input框的 focus事件，获得焦点时修改父级边框颜色。

没错，但这不是最优雅的答案。

## :focus-within伪类
如果使用`:focus-within`伪类, 只需要一句代码就可以实现上图效果

关键css代码
```css
.container:focus-within {
  border-color: #409EFF;
}
```
html结构如下
```html
<div class="container">
  <input type="text">
</div>
```
>到发稿时（2018.08.26）， `:focus-within`兼容性一般，如下所示
![:focus-within兼容性]({{site.baseurl}}/assets/images/2018/08/focus-within-compatible.png)

## 其它可能的应用场景
`:focus-within`还可能有一些其它应用场景，比如：
1. 表单沉浸式输入体验,也就是表单元素获得焦点时，隐藏其它干扰元素
2. 与 `:focus`配合，实现输入不同表单项时，表单显示不同背景或内容

大家充分发掘自己的脑洞吧！😃