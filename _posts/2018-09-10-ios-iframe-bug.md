---
title: 移动端纠结(一)：iframe在ios中的一些问题
description: 做前端开发的同学应该都知道，浏览器兼容性一直是一个令人忧虑的问题。以前是ie系列低版本的各种纠结, 现在移动端也好不到哪儿去。
categories:
 - mobile
tags:
 - ios
 - 前端
 - 移动端
 - iframe
---
# 前言
最近，工作上有一个需求。需要做一个移动端页面，页面分为三个部分:顶部视频区播放视频，中部引用第三方页面，底部下载按钮。大概长下面这个样子：
<div align="center">
<img src="{{site.baseurl}}/assets/images/2018/09/iframe.png" alt="demo图">
</div>

# 看起来很简单
看起来很简单，对吧？不就是上面`video`播放视频，中间使用`iframe`引入第三方页面，下面再放个按钮。

# 但是坑很多
`iframe`和`video`都有坑，`video`的坑不是今天的主题，以后再说，今天主要说说`iframe`在ios中(主要是safari及safari内核的系列浏览器)的坑(在 android下表现相对较好)。

# 第一坑: iframe在ios下无法滚动
正常情况下，如果不设置`scrolling="no"`,iframe引用页面内容超过iframe高宽，则会自动滚动。但是在ios无效，这并不是某个版本的问题，到本稿发稿时，ios最高版本为ios11.4，问题依然存在，解决的办法为：

给 iframe一个父级div, 给div设置如下样式：
```css
.container {
  -webkit-overflow-scrolling: touch;
  overflow-y: auto; /* 这里只需要纵向滚动，所以只写了 overflow-y */
}
```
html代码如下：
```html
<div class="container">
  <iframe src=""></iframe>
</div>
```

# 第二坑:iframe页面在ios上滚动时会闪烁，且会回弹到顶部
为了验证上面的代码的正确性，我引用了我司的移动端主页 <a href="https://m.sohu.com/" target="_blank">https://m.sohu.com/</a>。页面确实可以滚动了，想想有点小激动呢😉，于是就多滚动了几下，体验这喜悦之情。

结果，滚得快要哭出来了：
- 向下滚动时，被引用页面(`https://m.sohu.com`)有明显的闪烁，也就是有极短暂的白屏闪一下，然后又恢复正常，每次滚动几乎都会触发
- 在向下滚动到一定程度时，被引用页面(`https://m.sohu.com`)跳回到了顶部，类似刷新的效果

上面所提及的问题并不在所有页面体现——我做了一个简单的测试子页面`iframe.html`，上面只写了文字，将`iframe.html`的`body`的高度设置为`10000px`,以便页面能够滚动起来。将`iframe.html`引用到父页面之后滚动时并不会产生以上情况。

经过多方的搜索，产生以上问题的条件大概有以下几点:
- 一般情况下，当被引用页面高度超过iframe高度2倍或以上时会有可能出现以上问题
- 当被引用的页面在滚动时对dom进行了增加，删除或者样式的更改，有可能会出现以上问题

这是一个棘手的问题，网上有参考办法:
> 给 iframe添加onload事件监听，在 onload 之后通过js获得被引用页面的body高度，将该高度值赋给iframe

但是这个办法在我这里行不通，因为我们这里引用的是第三方页面，受同源策略影响，我无法在父级页面获得子页面的高度。所以我没有对此办法进行验证。

这个问题我暂时没辙，期待苹果公司尽快解决吧。如果大家谁有好的解决办法，欢迎交流。QQ: 187351549

# 第三坑: iframe宽度在ios会被内容撑开，无论设置宽度是多少
同样的，这个问题也出现在引用`https://m.sohu.com`时，我自己做的测试页面`iframe.html`则没有出现这种情况，原因暂时不明。

解决思路如下:

- 将 iframe 设置属性`scrolling="no"`,注意要判断平台是`ios`才添加该属性，否则该属性会导致在android下iframe无法滚动,这是关键性一步:
```html
  <iframe src="" scrolling="no"></iframe>
```
- 设置 iframe 样式(其实样式不是必需的，在我本机测试不添加样式也可以正常展示，但为了通用和保险，加上为好):
```css
iframe {
    width: 100%;
    max-width: 100%;
}
```

# 第四坑: iframe页面的meta:viewport会继承父页面
做过移动端开发的同学应该比较清楚，一般移动端页面都会添加一个meta:viewport标签：
```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
```
我们为了在各机型上表现一致，有时候会通过js动态修改这个值。业界比较有名的有<a href="https://github.com/amfe/lib-flexible" target="_blank">淘宝flexible方案</a>,它会根据设备dpr的不同而设置不同的缩放规则。

这个时候，如果子页面的缩放规则与父页面不同，则强制按照父页面规则进行缩放，所以实际表现有可能会被放大或者缩小，达不到预期的效果。

这个事情相当蛋疼，设置什么值都不合适，只要父子页面规则不匹配，就会有问题。

# 后记
在解决以上问题的过程中，还搜索到一些可能出现的问题，但是未在我们当前场景中出现，也一并记录:
- 被iframe引用的页面中如果有`position:fixed;`的元素，则会失效
- 被iframe引用的页面中的a锚点失效
- iframe页面切换的时候，切换后的页面样式莫名变大


# 结语
因为跨域的情况，给我们解决问题带来了不小的麻烦。如果要解决，可以考虑如下方法：
- 实施同源方案，比如将页面内容抓取到同源服务器，定时同步，或者开放接口让第三方文件上传至同源服务器。这两种方法成本都比较高，在不得已的情况下可以考虑。
- 寄希望于ios开发团队解决问题。

目前暂时发现iframe这些问题，移动端还有一些其它的问题,如 video, 事件等，回头有时间再出续篇。

> 工作不易，且行且珍惜。