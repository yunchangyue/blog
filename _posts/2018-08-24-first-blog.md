---
layout: default
title: 开门第一篇-我的博客是怎样炼成的
---

# {{page.title}}
> 随着向中年油腻男逐渐靠近，脑容量好像越来越小。这才想起小学的至理名言——
>"好记性不如烂笔头"

## 在哪写？

本来想，作为一个全干工程师，博客必须要自己手写、自己搭建才够风骚啊。后来想想，时间太长了，等出来菜都凉了——要不，找个模板吧——那还得管服务器呢。

不如干脆“回娘家”——就用github pages,免费服务器，无限流量，挺好。

## 那就愉快的开始吧
使用Jekyll（发音/'dʒiːk əl/，"杰克尔"），它是一个静态站点生成器，它会根据网页源码生成静态文件。
>我用的是mac,下面的操作都是基于macOS平台的

### 1. 安装ruby 和 Jekyll
由于 mac 系统自带ruby，所以不用安装ruby，直接开始安装 jekyll
```shell
gem install jekyll
```
愉快地等待安装完成，但是，出错了：
{% highlight shell %}
ERROR:  Could not find a valid gem 'jekyll' (>= 0), here is why:
          Unable to download data from https://rubygems.org/ - SSL_connect returned=1 errno=0 state=SSLv2/v3 read server hello A: tlsv1 alert protocol version (https://rubygems.org/latest_specs.4.8.gz)
{% endhighlight %}
看来起来像是被墙了，所以需要翻墙或者找一个国内镜像。在网上好一顿找，在试验了很多错误地址，包括淘宝镜像之后，最终发现可用地址 https://gems.ruby-china.com/
下面开始换掉镜像地址：
```shell
 gem sources --remove https://rubygems.org/
 gem sources -a https://gems.ruby-china.com/
```
然后开始安装jekyll:
```shell
gem install jekyll
```
我x,终于不报错了，结果，话还没说完就蹦个错误：
```shell
Fetching: public_suffix-3.0.2.gem (100%)
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.0.0 directory.
```
没有目录写权限，ok，那就提升到管理员
```shell
sudo gem install jekyll
```
又给我来一错：
```shell
ERROR:  Error installing jekyll:
	public_suffix requires Ruby version >= 2.1.
```
ruby版本低了,
升级ruby。通过 rvm 管理器升级,先安装rvm,如果已经安装过的话，就不用安装了(可以通过rvm -v查看版本号，如果没有 rvm命令，表示没有安装)：
```shell
curl -L get.rvm.io | bash -s stable
```
一路安装，生怕出错，还好。安装完成之后，会有提示(可能每个人的路径不一样)：
```
To start using RVM you need to run `source /Users/leyunchang/.rvm/scripts/rvm`
```
那就按照提示来吧：
```shell
source /Users/leyunchang/.rvm/scripts/rvm
```
安装完了查看一下版本：
```shell
rvm -v
```
看到提示：
```shell
rvm 1.29.4 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
```
说明安装成功了，那就安装一个比较高版本的。如果不知道安装哪个版本,可以使用：
```shell
rvm list known
```
看一下有哪些版本，本博客发表时最高版本为2.6.0,那就安装2.5.1吧:
```shell
rvm install 2.5.1
```
安装的时间有点长，静静的等待。

安装完成之后，就可以愉快的安装jekyll了。
```shell
sudo gem install jekyll
```
输入管理员密码，一路畅通的安装了。安装完成最后会有提示
```shell
Done installing documentation for public_suffix, addressable, colorator, http_parser.rb, eventmachine, em-websocket, concurrent-ruby, i18n, rb-fsevent, ffi, rb-inotify, sass-listen, sass, jekyll-sass-converter, ruby_dep, listen, jekyll-watch, kramdown, liquid, mercenary, forwardable-extended, pathutil, rouge, safe_yaml, jekyll after 43 seconds
25 gems installed
```
### 2.开始搭建博客
参考了阮一峰老师的文章，说得比较详细，这里就不再啰嗦了。

传送门： <a target="_blank" href="http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html">搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门</a>
### 3.通过 jekyll 构建运行
```shell
jekyll build
```