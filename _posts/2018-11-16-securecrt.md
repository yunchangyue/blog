---
title: 记一次secureCRT中文乱码解决过程
description: 一直觉得macbook终端工具不错，但又觉得连远程服务器很麻烦。想找一款终端工具，可以记住远程服务器的连接。找来找去，觉得secureCRT还行，使用过程中有一些乱码的坑，在这里记录一下填坑过程。
categories:
 - tools
tags:
 - secureCRT
 - 工具
 - 终端
---
# 一、什么是secureCRT?
`secureCRT`是一款终端软件，像`windows`中的`xShell`软件一样， 在 macbook 上基本可以取代系统自带的终端工具，这个软件也有`windows`版本。

# 二、为什么是secureCRT?
其实我用自带的终端工具已经有一段时间，觉得并没有什么不适。后来手里的服务器多了，记住密码是一个问题。自带的终端工具，经过我的研究，好像只能做到记住服务器，不能记住密码自动连接。一个个连接实在是太费精力，所以就想找一款像 `windows`平台上的`xShell`的软件，来自动记住我的服务器用户名密码，点击某个服务器时就可以自动连接，不用每次连接时都输入密码。找了很多款终端软件，像`ShellCraft`,`SSH Shell`,`Zoc Terminal`等等,总是不尽人意，最后找到了`secureCRT`,觉得还可以，下载量也多，就决定用它了。

# 三、中文乱码问题
之前在自带终端里是没有过中文乱码问题的。在我使用secureCRT的过程中，出现了乱码问题。
> 最开始的时候，以为只是软件编码问题。但其实这里乱码问题分几类，分别对应不同的解决办法，以下会有描述。

### 1.`git log`,`git diff`中的乱码
如图所示：
![git 乱码]({{site.baseurl}}/assets/images/2018/11/git.png)

在网上找了很多教程，普遍的说法是：
- 设置secureCRT的编码为UTF-8
![git 乱码]({{site.baseurl}}/assets/images/2018/11/config.png)
![编码设置为utf-8]({{site.baseurl}}/assets/images/2018/11/charset.png)

但我发现设置之后并没有效果。

后来我发现使用`cat`命令查看 git diff 中的某个差异文件的时候，却没有乱码。这个时候机智的我已猜到，这可能是`git`的编码问题,所以在git配置中修改配置文件设置编码：
```shell
vi ~/.gitconfig
```
添加配置项如下：
```gitconfig
[core]
	quotepath = false
[gui]
	encoding = utf-8
[i18n]
	commitencoding = utf-8
	logoutputencoding = utf-8
[svn]
	pathnameenoding = utf-8
```

保存退出。信心满满，觉得大功要告成啦！

但现实总是残酷的——并没有什么作用！

实际上，`git diff` 和 `git log` 是通过 `less` 进行输出的。所以配置`less`字符编码就可以:
```shell
export LESSCHARSET=UTF-8
```
> **重点**：网上有很多攻略上说,在 /etc/profile 文件里面增加上面的代码。但我这里亲测无用，直接在命令行里执行上面这句话才会成功。

此时，再执行`git log`, `git diff`，完美显示中文。

### 2.`vi`,`vim`中的乱码
本以为再无问题。后来无意中`vim`一个文件的时候，又出现乱码了！

这个时候，需要确定是否字符集的问题。vim文件之后使用`:set encoding=utf8`看看是否能显示正常。如果能显示正常基本就能确认是字符编码的问题。

这个时候，解决办法就是更改vim环境变量：

- 使用 vi ~/.vimrc (有些同学电脑上可能没有这个文件，如果没有，新建这个文件即可)
- 在文件中添加代码 set encoding=utf-8 fileencodings=ut-8,ucs-bom,p936,big5
- 保存退出

这个时候再 vim 打开文件就没有乱码问题了。

