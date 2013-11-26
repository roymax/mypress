---
layout: post
title: "Auto sync file with Nitrous.io on Mavericks"
date: 2013-11-15 19:02
comments: true
external-url: 
published: true
categories: ['nitrous.io']
---

自从升级到 Mavericks 后，Nitrous.IO 的 [OSX App](https://www.nitrous.io/mac) 就失效了，官方的最新版本还没有释放出来，这导致离线编辑自动同步到线上测试变得不可用。

我知道有rsync同步的方法，但一直也懒得去研究，直到我遇到[这篇文章](http://notes.geoffreyd.me/local-file-editing-with-nitrous-io-on-mavericks) 后算是解决问题了，只需要一条命令。

开始前需要安装`fswatch`和`unison`，前者用于扫描文件变化，后者用于双向实时同步（跨Win/Linux/OSX），而且支持通过 SSH/RSH/Socket 进行同步。如果你也像我一样在使用[brew](/blog/the_homebrew_better_than_the_macports)，可以通过命令 `brew install fswatch unison`安装。

然后进入 `~/Nitrous.IO/gobox/bin` 创建一个同步脚本，我照抄上面的文章

```
#!/bin/bash
fswatch . "date +%H:%M:%S; unison /Users/roymax/Nitrous.IO/gobox/ ssh://action@gobox//home/action/workspace -ui text -terse -batch"
```

当我们需要工作时，只需要进入目录执行`./bin/start_sync.sh`，当本地编辑文件时将自己上传到 Nitrous.IO 的对应目录了。

不过据本人测试，同步有点延时但总算能够正常工作。

