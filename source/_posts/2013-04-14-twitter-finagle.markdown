---
layout: post
title: "twitter finagle"
date: 2013-04-14 11:28
comments: true
external-url: 
published: true
categories: [scala,java,finagle]
---

## 简介

Finagle是twitter内部的一个非常重要的核心工具库，它大量被应用于twitter内部系统。Finagle是一个异步的网络框架，能被用于实现异步的RPC客户端程序和基于JVM语言体系的服务端，包括Java，Scala，Jython等。

它还支持众多协议

- HTTP
- HTTP Streaming (Comet)
- Thrift
- Memcached/Kestrel
- MySQL
- protobuf
- ......

基于Finagle的Server端特性支持

- Backpressure (to defend against abusive clients)
- Service Registration (e.g., via Zookeeper)
- Distributed Tracing
- OpenSSL

基于Finagle的Client端特性支持

- Connection Pooling
- Load Balancing
- Failure Detection
- Failover/Retry
- Distributed Tracing (à la Dapper)
- Service Discovery (e.g., via Zookeeper)
- Rich Statistics
- Native OpenSSL Bindings

twitter是Scala重度用户，同时Finagle也是写于Scala，围绕Finagle衍生了很多辅助的项目

- [scrooge](https://github.com/twitter/scrooge)
- [sbt-scrooge](https://github.com/twitter/sbt-scrooge)
- [scala-bootstrapper](https://github.com/twitter/scala-boostrapper)
- [sbt-package-dist](https//github.com/twitter/sbt-package-dist)
- ......

其中`scrooge`是一个**thrift**代码生成器的Scala版本，是*Apapche Thrift*版本的替换且保持与它兼容，它同时用于生成符合*Finagle*使用的代码，当前只支持生成Scala和Java文件。而`sbt-scrooge`则是一个sbt插件。

`scala-bootstrapper`类似于一个脚手架工具，用于生成一个基础的`finagle`项目

`sbt-package-dist`则是一个基于sbt的打包插件

## Hello World

这个`Hello World`将由*scala-bootstrapper*项目入手，该项目是一个ruby项目，因此确保有ruby安装环境。

通过`scala-bootstrapper <project>`可以生成项目，但必须先建立一个项目目录，否则在当前目录创建


```
$ mkdir myfinagle && cd myfinagle
$ scala-bootstrapper myfinagle
writing Capfile
writing config/development.scala
writing config/production.scala
writing config/staging.scala
writing config/test.scala
writing Gemfile
writing project/build/MyfinagleProject.scala
writing project/build.properties
writing project/plugins/Plugins.scala
writing run
writing src/main/scala/com/twitter/myfinagle/MyfinagleServiceImpl.scala
writing src/main/scala/com/twitter/myfinagle/config/MyfinagleServiceConfig.scala
writing src/main/scala/com/twitter/myfinagle/Main.scala
writing src/main/thrift/myfinagle.thrift
writing src/scripts/console
writing src/scripts/startup.sh
writing src/test/scala/com/twitter/myfinagle/AbstractSpec.scala
writing src/test/scala/com/twitter/myfinagle/MyfinagleServiceSpec.scala
```

我还根据需要，修改了一些依赖配置


```
.....

libraryDependencies ++= Seq(
  "org.scala-lang" % "jline"                 % “2.9.1”,  #升级版本
  "com.twitter"    % "scrooge-generator"     % "3.0.8" intransitive(),  #升级版本
  "com.twitter"    % "scrooge-runtime"       % "3.0.8" intransitive(),  #升级版本
  "com.twitter"    % "finagle-core"          % “6.1.2”,  #升级版本
  "com.twitter"    % "finagle-thrift"        % “6.1.2”,  #升级版本 
  "com.twitter"    % "finagle-ostrich4"      % “6.1.2”,  #升级版本
  "org.scalatest" %% "scalatest"             % "1.7.1" % "test",
  "com.twitter"   %% "scalatest-mixins"      % "1.1.0" % "test"
)

...

CompileThriftScrooge.scroogeVersion := "3.0.8"  #升级版本

```

升级版本后，需要打开`src/main/scala/com/twitter/<project>/FinalgeServiceImpl.scala`将最后的`shutdown()`方法删除，否则无法编译通过

最后按照项目目录下的`TUTORIAL.md`执行更新和测试启动是否成功

但要正常运行，请看有哪些坑吧

## 坑

### sbt

- 执行`sbt update`前，如果你有自己架设的*maven代理服务器*，建议添加一下代理仓库，或加到项目根目录下的`sbt`文件里

```
  #!/bin/bash

  export SBT_PROXY_REPO="proxy server address"
```

- 如果打算使用自己的maven代理，将`project/plugins.sbt`里的**resolvers**部分指定的仓库加入到私有仓库。

- twitter的仓库(http://maven.twttr.com/)已被污染了IP，请确认你能访问该域名，否则应该修改`/etc/hosts`添加`199.59.148.212 maven.twttr.com`

- `freemarker`也无法访问，好像无关痛痒，注释掉

- 没有必要单独安装sbt和scala，因为项目根目录本来就有一个sbt的脚本，它会自动下载并构建一个sbt和scala的运行环境，并且推荐使用该脚本文件

- 如果需要使用本地的sbt环境，需要指定sbt版本`echo "sbt.version=0.11.3" > ./project/build.properties`


### scala-bootstrapper

尝试`gem install scala-bootstrapper`发现可以安装，观察一下版本发现有点旧了，最后还是自己编译安装

```
$ git clone https://github.com/twitter/scala-bootstrapper.git
$ cd scala-bootstrapper
$ rake build
$ gem install pkg/scala-bootstrapper-*.gem
```

但我在`rake build`这一步时遇到问题

  fail "ERROR: 'rake/rdoctask' is obsolete and no longer supported. Use 'rdoc/task' (available in RDoc 2.4.2+) instead."

最后通过修改源代码解决

```
# Add this line just before the require to fix the deprecation notice
$LOAD_PATH.sort_by! { |p| p.match(%r[/rake\-]) ? 1 : 0 }

# 找到rake/rdoctask 替换为 rdoc/task
require 'rdoc/task'
```

### scrooge 

编译脚本会从twitter的仓库下载一个[scrooge-3.0.0.zip](http://maven.twttr.com/com/twitter/scrooge/3.0.0/)的压缩包，不知道什么原因，scrooge的最新版本没有上传到仓库，同时也找不到这个最新版本的zip。那么如果需要使用新版本的scrooge，那要么使用`maven`，要么自己打一个zip包放到本地仓库

我不知道twitter内部的计划，我通过以下方法生成了一个`3.0.8`版本

```
cd /tmp
git clone git://github.com/twitter/scrooge.git
cd scrooge 
mvn package dependency:copy-dependencies

cd /tmp
wget http://maven.twttr.com/com/twitter/scrooge/3.0.1/scrooge-3.0.1.zip
unzip scrooge-3.0.1
mv scrooge-3.0.1 scrooge-3.0.8
cd scrooge-3.0.8 
rm -rf *.jar libs/*.jar
sed -i "" -e "s/3.0.1/3.0.8/g" ./scripts/scrooge
cp /tmp/scrooge/scrooge-generator/target/scrooge-generator-3.0.8.jar ./scrooge-3.0.8.jar
cp -r /tmp/scrooge/scrooge-generator/target/dependency/*.jar libs/
cd ..
zip -r scrooge-3.0.8.zip scrooge-3.0.8
cp scrooge-3.0.8.zip ~/.m2/repository/com/twitter/scrooge/3.0.8/
```

## 后记

`Finagle`作为twitter的核心工具库，性能和稳定性应该还是有一定保障的，而且从公布开源的出来的一系列工具堆栈非常有吸引力，如用于数据收集和报告的ostrich，性能分析的zipkin等。但文档确认稍候欠奉，会遇上各种坑。

再加上Scala的各版本不太兼容，稍不注意你就找不到对应的包。比如我希望设置用于scala-2.10.0版本就遇到些问题，而且花了好些时间才解决。估计scala发行版本再不保障版本的向前兼容，估计以后也很难普及。scala的社区个人觉得还是不够好，关注的人太少。scala在github上1196人加星，374人fork；而ruby的数据则是3826/867。

另不知基于什么原因，估计twitter已经开始将sbt迁移到maven，以至于有多个sbt插件项目已经至少半年以上没有更新了，其中就包括了`scala-bootstrapper`和`sbt-scrooge`。所以，如果确定要使用finagle，建议还是使用maven吧，反正`scala-bootstrapper`生成的代码也没帮到多少，自己用[giter8](https://github.com/n8han/giter8)也不是什么难事。

## 参考资料

- [Finagle+Spring](http://sunwenfeng.blogspot.com/2012/12/twitter-finagle-scalarpc.html)
- [https://github.com/twitter/scala_school](https://github.com/twitter/scala_school)
- [Effective scala](http://twitter.github.io/effectivescala/index-cn.html)
- [使用SBT构建Scala应用](https://github.com/fujohnwang/real_world_scala/blob/master/02_sbt.markdown)