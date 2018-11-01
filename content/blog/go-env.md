---
title: "Go语言环境搭建详解"
date: 2018-10-31T12:29:40+06:00
type: post
image: images/blog/go-env/header.jpg
author: 邓冰寒
tags: ["Go", "Golang", "development", "environment"]
---

## 起源与发展

Go 语言起源 2007 年，并于 2009 年正式对外发布。它从 2009 年 9 月 21 日开始作为谷歌公司 20% 兼职项目，即相关员工利用 20% 的空余时间来参与 Go 语言的研发工作。该项目的三位领导者均是著名的 IT 工程师：Robert Griesemer，参与开发 Java HotSpot 虚拟机；Rob Pike，Go 语言项目总负责人，贝尔实验室 Unix 团队成员，参与的项目包括 Plan 9，Inferno 操作系统和 Limbo 编程语言；Ken Thompson，贝尔实验室 Unix 团队成员，C 语言、Unix 和 Plan 9 的创始人之一，与 Rob Pike 共同开发了 UTF-8 字符集规范。自 2008 年 1 月起，Ken Thompson 就开始研发一款以 C 语言为目标结果的编译器来拓展 Go 语言的设计思想。

在 2008 年年中，Go 语言的设计工作接近尾声，一些员工开始以全职工作状态投入到这个项目的编译器和运行实现上。Ian Lance Taylor 也加入到了开发团队中，并于 2008 年 5 月创建了一个 gcc 前端。

Russ Cox 加入开发团队后着手语言和类库方面的开发，也就是 Go 语言的标准包。在 2009 年 10 月 30 日，Rob Pike 以 Google Techtalk 的形式第一次向人们宣告了 Go 语言的存在。

直到 2009 年 11 月 10 日，开发团队将 Go 语言项目以 BSD-style 授权（完全开源）正式公布了 Linux 和 Mac OS X 平台上的版本。Hector Chu 于同年 11 月 22 日公布了 Windows 版本。

作为一个开源项目，Go 语言借助开源社区的有生力量达到快速地发展，并吸引更多的开发者来使用并改善它。自该开源项目发布以来，超过 200 名非谷歌员工的贡献者对 Go 语言核心部分提交了超过 1000 个修改建议。在过去的 18 个月里，又有 150 开发者贡献了新的核心代码。这俨然形成了世界上最大的开源团队，并使该项目跻身 Ohloh 前 2% 的行列。大约在 2011 年 4 月 10 日，谷歌开始抽调员工进入全职开发 Go 语言项目。开源化的语言显然能够让更多的开发者参与其中并加速它的发展速度。Andrew Gerrand 在 2010 年加入到开发团队中成为共同开发者与支持者。

在 Go 语言在 2010 年 1 月 8 日被 Tiobe（闻名于它的编程语言流行程度排名）宣布为 “2009 年年度语言” 后，引起各界很大的反响。目前 Go 语言在这项排名中的最高记录是在 2017 年 1 月创下的第13名，流行程度 2.325%。

## Go发展历程

* 2007 年 9 月 21 日：雏形设计
* 2009 年 11 月 10日：首次公开发布
* 2010 年 1 月 8 日：当选 2009 年年度语言
* 2010 年 5 月：谷歌投入使用
* 2011 年 5 月 5 日：Google App Engine 支持 Go 语言
  
从 2010 年 5 月起，谷歌开始将 Go 语言投入到后端基础设施的实际开发中，例如开发用于管理后端复杂环境的项目。有句话叫 “吃你自己的狗食”，这也体现了谷歌确实想要投资这门语言，并认为它是有生产价值的。

Go 语言的官方网站是 golang.org，这个站点采用 Python 作为前端，并且使用 Go 语言自带的工具 godoc 运行在 Google App Engine 上来作为 Web 服务器提供文本内容。在官网的首页有一个功能叫做 Go Playground，是一个 Go 代码的简单编辑器的沙盒，它可以在没有安装 Go 语言的情况下在你的浏览器中编译并运行 Go，它提供了一些示例，其中包括国际惯例 “Hello, World!”。

更多的信息详见 github.com/golang/go，Go 项目 Bug 追踪和功能预期详见 github.com/golang/go/issues。

Go 通过以下的 Logo 来展示它的速度，并以囊地鼠（Gopher）作为它的吉祥物。

![go-log](/images/blog/go-env/go-logo.jpg)

谷歌邮件列表 golang-nuts 非常活跃，每天的讨论和问题解答数以百计。

关于 Go 语言在 Google App Engine 的应用，这里有一个单独的邮件列表 google-appengine-go，不过 2 个邮件列表的讨论内容并不是分得很清楚，都会涉及到相关的话题。go-lang.cat-v.org/ 是 Go 语言开发社区的资源站，irc.freenode.net 的#go-nuts 是官方的 Go IRC 频道。

@golang 是 Go 语言在 Twitter 的官方帐号，大家一般使用 #golang 作为话题标签。

这里还有一个在 Linked-in 的小组：www.linkedin.com/groups?gid=2524765&trk=myg_ugrp_ovr。

Go 编程语言的维基百科：en.wikipedia.org/wiki/Go_(programming_language)

Go 语言相关资源的搜索引擎页面：gowalker.org

Go 语言还有一个运行在 Google App Engine 上的 Go Tour，你也可以通过执行命令 go install go-tour.googlecode.com/hg/gotour 安装到你的本地机器上。对于中文读者，可以访问该指南的 中文版本，或通过命令 go install https://bitbucket.org/mikespook/go-tour-zh/gotour 进行安装。

## 搭建开发环境

如果你打算按照官方[官方入门指南](https://golang.org/doc/install)来安装，可跳过本教程。(注：文中有`${VERSION}`的地方表示你将要安装Go的版本，如你要使用 `go1.10` 则 `${VERSION}` 为 `1.10`)

本教程介绍介绍三种Go语言开发环境安装方法：

1. Go源码安装
2. Go标准包安装
3. 第三方工具安装

## Go源码安装

对于一般开发者而言，推荐`Go标准包安装`，直接跳过这个环节。当然，如果你有兴趣和需求，想要通过Go源码安装，那就继续往下看：

### 前提条件

>重要提示： 由于众所周知的原因，在安装前请自行准备梯子(关键字：shadowsocks, proxifier)，否则可能下载不到安装包。

* Mac操作系统：需要安装Xcode
* Linux操作系统：需要安装gcc等工具。在Ubuntu发行版可通过在终端中执行sudo apt-get install gcc libc6-dev来安装编译工具。对于Fedora发行版则使用yum install install gcc libc6-dev来安装。
* Windows操作系统：需要安装MinGW，然后通过MinGW安装gcc，并设置相应的环境变量。

在你确定安装方式后，到[官网](https://golang.org/dl/)下载相应的 `go${VERSION}.src.tar.gz`, 下载后解压缩到你的工作目录，执行如下代码：

```bash
cd go/src
./all.bash
```

当出现 `ALL TESTS PASSED` 即表示安装成功。

接下来在 `.bashrc` 或 `.zshrc` 设置以下环境变量。

```bash
export GOPATH=$HOME/gopath
export PATH=$PATH:$HOME/go/bin:$GOPATH/bin
```

你想要重启终端，让设置生效。在终端下输入命令`go`来验证是否安装成功。

![go-cmd](/images/blog/go-env/go-cmd.png)

## Go标准包安装

Go提供了每个平台打好包的一键安装，这些包默认会安装到如下目录：/usr/local/go (Windows系统：c:\Go)，当然你可以改变他们的安装位置，但是改变之后你必须在你的环境变量中设置如下信息：

```bash
export GOROOT=$HOME/go  
export GOPATH=$HOME/workspace/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

以上命令对于Mac和Unix用户来说可以写入.bashrc或者.zshrc文件，对于windows用户来说通过图形界面配置环境变量.

![go-cmd](/images/blog/go-env/win-env-step3.png)

### Mac 操作系统

访问[下载地址](http://golang.org/dl/)，选择64位系统下载`go${VERSION}.darwin-amd64.pkg`，双击下载文件，一路默认安装点击下一步，这个时候go已经安装到你的系统中，默认已经在PATH中增加了相应的路径,这个时候打开终端，输入`go`

![go-cmd](/images/blog/go-env/go-cmd.png)

如果出现图中go的Usage信息，那么说明go已经安装成功了；如果出现该命令不存在，那么可以检查一下自己的PATH环境变中是否包含了go的安装目录。

### Linux 操作系统

访问[下载地址](http://golang.org/dl/)，32位系统下载`go${VERSION}.linux-386.tar.gz`，64位系统下载`go${VERSION}.linux-amd64.tar.gz`，

假定你想要安装Go的目录为 `${GO_INSTALL_DIR}`，后面替换为相应的目录路径。

解压缩tar.gz包到安装目录下：`tar zxvf go1.8.3.linux-amd64.tar.gz -C ${GO_INSTALL_DIR}`。

设置PATH，`export PATH=$PATH:$GO_INSTALL_DIR/go/bin`

Linux系统下安装成功之后执行go显示的信息如下：

![go-cmd](/images/blog/go-env/go-cmd-linux.png)

### Windows 操作系统

访问[下载地址](http://golang.org/dl/)，32位请选择名称中包含 windows-386 的 msi 安装包，64 位请选择名称中包含 windows-amd64 的。下载好后运行，不要修改默认安装目录 C:\Go\，若安装到其他位置会导致不能执行自己所编写的 Go 代码。安装完成后默认会在环境变量 Path 后添加 Go 安装目录下的 bin 目录 C:\Go\bin\，并添加环境变量 GOROOT，值为 Go 安装根目录 C:\Go\ 。

#### 验证安装是非成功

在运行中输入 cmd 打开命令行工具，在提示符下输入 go，检查是否能看到 Usage 信息。输入 cd %GOROOT%，看是否能进入 Go 安装目录。若都成功，说明安装成功。

![go-cmd](/images/blog/go-env/go-cmd-win.png)

## 第三方工具GVM安装

为什么要安装第三方工具，理由如下：

1. 管理 Go 的多个版本，包括安装、卸载和指定使用 Go 的某个版本
2. 查看官方所有可用的 Go 版本，同时可以查看本地已安装和默认使用的 Go 版本
3. 管理多个 GOPATH，并可编辑 Go 的环境变量
4. 可将当前目录关联到 GOPATH
5. 可以查看 GOROOT 下的文件差异

```bash
bash <<(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)  
```

### 使用 GVM

直接输入 gvm，查看使用帮助

```bash
Usage: gvm [command]

Description:
  GVM is the Go Version Manager

Commands:
  version    - print the gvm version number
  get        - gets the latest code (for debugging)
  use        - select a go version to use (--default to set permanently)
  diff       - view changes to Go root
  help       - display this usage text
  implode    - completely remove gvm
  install    - install go versions
  uninstall  - uninstall go versions
  cross      - install go cross compilers
  linkthis   - link this directory into GOPATH
  list       - list installed go versions
  listall    - list available versions
  alias      - manage go version aliases
  pkgset     - manage go packages sets
  pkgenv     - edit the environment for a package set
```

### 安装指定的 Go 版本

```bash
gvm install go1.10
```

它背后做的事情是先把源码下载下来，再用 C 做编译。安装好之后，指定默认使用这个版本，加上 --default 即可，省去每次敲 gvm use

```bash
gvm use go1.10 --default  
```

这个时候查看 go 环境变量

```bash
go env
```

应该显示正确的GOROOT，GOPATH等

```bash
GOARCH="amd64"
GOBIN=""
GOCACHE="/Users/johnd/Library/Caches/go-build"
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/johnd/.gvm/pkgsets/go1.10/hidevops:/Users/johnd/go:/Users/johnd/.gvm/pkgsets/go1.10/global"
GORACE=""
GOROOT="/Users/johnd/.gvm/gos/go1.10"
GOTMPDIR=""
GOTOOLDIR="/Users/johnd/.gvm/gos/go1.10/pkg/tool/darwin_amd64"
GCCGO="gccgo"
CC="clang"
CXX="clang++"
CGO_ENABLED="1"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/zw/ynx11jnx6j5gwbhkqqdqf0wm0000gn/T/go-build844998127=/tmp/go-build -gno-record-gcc-switches -fno-common"
```

### 管理多个 GOPATH

GVM 还一个很实用的功能，可以管理多个 GOPATH（Go package），通过 gvm pkgset 命令, 可以查看使用帮助。

```bash
gvm pkgset
```

```bash
= gvm pkgset

* http://github.com/moovweb/gvm

== DESCRIPTION:

GVM pkgset is used to manage various Go packages

== Usage

  gvm pkgset Command

== Command

  create     - create a new package set
  delete     - delete a package set
  use        - select where gb and goinstall target and link
  empty      - remove all code and compiled binaries from package set
  list       - list installed go packages
```

### 安装IDE

Go语言开发推荐使用[Visual Studio Code](https://code.visualstudio.com/)或[Goland](https://www.jetbrains.com/go/)，具体安装方法可参考对应官方指南。

如果你选择Visual Studio Code，需要额外安装支持Go语言等相关插件

![go-plugins](/images/blog/go-env/go-vscode-plugins.png)

## 重要参考书籍资料

[Effective Go中文版](https://www.kancloud.cn/kancloud/effective/72199)

以上就绪，接下来就可以写Go的代码了。