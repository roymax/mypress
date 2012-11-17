---
layout: post
title: "problems with meteor"
date: 2012-11-17 12:04
comments: true
categories: [meteorjs]
---

最近一个月关注了一下几个前端MVC库，包括有Emberjs/Angularjs/Meteorjs/Backbonejs等，发现所谓的前端后端的界线越来越模糊了。

Angularjs非常吸引人，看起来真的不错，而且还是由google这个业界巨头开源出来，质量值得信任。如果已经有一个良好的后端架构支持，Angularjs/Backbonejs之间必定任选其一都不会错，说到底就是看用哪家的API更舒服一些了。

如果从零开始学习或玩票性质地想做一些尝试性项目，我个人推崇Meteor。这个家伙启动成本真的很低很低，一体化的框架，基于nodejs+mongodb，社区也足够的活跃。当然了，优点也就是缺点，Full Stack Framework就是意味着一些制约了。

Emberjs则有点像Rails，或者直接说就是一个完全遵循Rails思想的框架，而且天生就跟Rails做高整合。如果熟悉Rails并且有一个基于Rails的后端或许是一个不错的选择，但个人感觉学习曲线有点徒（当然这更多是臆测，因为个人上手感觉不大习惯）。

另外还有几个Knockout/CanJS/Batman/Spine就没细看了，从github上的fork和wacth来看，感觉社区活跃度都比不上前面提到的几家。

最后尝试用[meteor](http://www.meteor.com)写一个app，遇到一些小问题并记录下来

## 加载nodejs package

Meteor实际上基于nodejs，所以可以直接加载nodejs已有的package

```
$ meteor create myapp
$ cd myapp/.meteor/local/build/server
$ npm install <package>
```

然后用以下方法加载

```
var pkg = __meteor_bootstrap__.require('package_name');
```


## 项目结构

Meteor 有两种方式拆分服务端和客户端代码

1. 使用`Meteor.is_client`和`Meteor.is_server`区别代码块，`meteor create project`后生成的代码就是用这种方法。
2. 将服务端和客户端代码分别放到项目根目录下的`server`和`client`文件夹。

使用第二种方式，需要在项目目录下新建`server`和`client`目录，并将Meteor生成的项目的主文件分别拆分到目录，分别命名为`${ROOT}/server/proj_server.js`和`${ROOT}/client/proj_client.js`。

另外，在项目根目录下的js文件可以设置公共的方法和变量，它能同时被server和client端调用。

## 使用Meteor.http特性

因为文档看得不够仔细，总是得到一个undefined的错误，后来发现Meteor.http也是Packages之一，所以需要`meteor add http`才可以使用。

## 关于编码

如果需要通过`Meteor.http.get`方法获取non-UTF8的内容就需要注意，因为Meteor将response设置成 UTF8返回，并不可以指定encoding。如果确认需要这处理GBK这个字符集，那么需要一点hacking code

修改`packages/http/httpcall_server.js`文件

```
     var req_options = {
       url: new_url,
       method: method,
-      encoding: "utf8",
+      encoding: options.encoding || "utf8",
       jar: false,
       timeout: options.timeout,
       body: content,
```

然后，你可以这样做

```
  Meteor.methods({getMyFunds: function() {
    this.unblock();
    var result = Meteor.http.get("http://hq.sinajs.cn/list=of110002"
      , { encoding: "base64" }
      );
    if (result.statusCode === 200) {
        var require = __meteor_bootstrap__.require;
        var Iconv = require('iconv').Iconv;
        var iconv = new Iconv('GBK', 'UTF-8//TRANSLIT//IGNORE');
        return iconv.convert(new Buffer(result.content,'base64')).toString();
    }

  }});
```