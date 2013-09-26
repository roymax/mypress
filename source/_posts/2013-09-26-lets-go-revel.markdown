---
layout: post
title: "let's go - revel"
date: 2013-09-26 23:05
comments: true
external-url: 
published: true
categories: ['nitrous.io', 'golang']
---

部署好[Nitrous.io的Go环境](/blog/2013/09/let-us-go/)和输入`Hello World!!`后，接下来试试如何开发一个Web应用，并同样输出`Hello World!!`，于是找到了[Revel](http://robfig.github.io/revel/)

## Revel

官网这样描述它**A high-productivity web framework for the Go language.**，而这里选择的原因只是因为看起来像`rails`罢了

那么开始吧，要先安装这个框架，照着官网「[Getting Started](http://robfig.github.io/revel/tutorial/gettingstarted.html)」

```
$ ssh gobox # 我配置了[ssh config](/blog/2013/09/let-us-go/)

(gobox) $ cd workspace
(gobox) $ go get github.com/robfig/revel/revel
```

稍候片刻安装完成，看看目录结构

```
.
├── bin
├── pkg
│   └── linux_amd64
└── src
    ├── code.google.com
    ├── github.com
```

多了几个目录，其中`src`是依赖库的源代码目录，`pkg`是编译后的代码目录，`bin`则是命令行工具，现在我们有了一个`revel`命令

```
~/workspace$ revel
~
~ revel! http://robfig.github.com/revel
~
usage: revel command [arguments]

The commands are:

    new         create a skeleton Revel application
    run         run a Revel application
    build       build a Revel application (e.g. for deployment)
    package     package a Revel application (e.g. for deployment)
    clean       clean a Revel application's temp files
    test        run all tests from the command-line

Use "revel help [command]" for more information.


```

> 如果想了解`Nitrous.io`默认给我们准备好的开发环境，可以打开`~/.bashrc`看看，里面包含了一些预设的环境变量。比如安装Go语言需要设置的`$GOPATH`和`$GOROOT`

## First Web App

```
$ revel new goweb #在src下会生成一个名goweb的项目
$ revel run goweb
~
~ revel! http://robfig.github.com/revel
~
INFO  2013/09/26 15:52:46 revel.go:292: Loaded module testrunner
INFO  2013/09/26 15:52:46 revel.go:292: Loaded module static
INFO  2013/09/26 15:52:46 run.go:57: Running goweb (goweb) in dev mode
INFO  2013/09/26 15:52:46 harness.go:157: Listening on :9000
```

这样就跑起来了，可以看到默认跑在`:9000`端口，但`Nitrous`只支持3000,4000,8080,8888这四个端口，那么先把端口改为`3000`

打开`~/workspace/src/goweb/conf/app.conf`找到第四行`http.port=9000`改为`http.port=3000`重新启动`revel`。

打开boxes列表找出[预览URL](http://help.nitrous.io/faq-localhost/)，加上3000端口从浏览器打开可以看到**It Works!**的大字

<img src="http://d.pr/i/tL3m+" alt="goweb it works">

> Tips: 域名很难记，但如果你像我一样在用MacBook，你还可以通过*Nitrous*的客户端打开`Port Forwarding`，然后就可以通过`http://localhost:<port>`来访问你的远端程序了

<img src="http://d.pr/i/s6zE+" alt="nitrous Port Forwarding">

注意：HTTPS貌似预览访问不了

## 再进一步

只是显示一个`It Works!`就结束还是单调了点，我打算传一个参数`name=roy`进去，然后显示`Hi,Roy`

用`Sublime Text`打开`~/Nitrous.IO/gobox`，使用`cmd+p`呼出搜索框输入`app.go`打开。

通过文档我知道，如果需要传入参数，我可以这么做

```
func (c App) Index(name string) revel.Result {
  return c.Render(name)
}

```

再打开`It Works!`所在的模板文件`app/views/App/Index.html`，同时修改为

```
...
<h1>hi \{\{.name\}\}!!</h1>
...#省略上下HTML部分
```

再刷新浏览器发现只显示出现`hi `，再把URL修改为`http://localhost:3000/?name=Roy`看看结果吧。

## 后记

观察一下`revel`生成的`goweb`项目，你会发现目录结构跟`rails`真的差不多，简单的摸索已经可以上手。接下来，或者可以试试连接数据库操作了。

-----

留下[Nitrous小尾巴](https://www.nitrous.io/join/YsusrqFe8dY)求闪电: [https://www.nitrous.io/join/YsusrqFe8dY](https://www.nitrous.io/join/YsusrqFe8dY)