---
layout: post
title: "node-restify 简介"
date: 2012-02-19 22:18
comments: true
published: true
categories: [技术]

---

[restify](https://github.com/mcavage/node-restify) 是[Node.js](http://nodejs.org)的模块。虽然`restify`的API或多或少的参考了[express](http://expressjs.com/)，但`restify`不是一个MVC框架，它是一套为了能够正确构建REST风格API而诞生的框架。

###安装restify

将`restify`安装到目录`restify-dmeo`
```
$ mkdir restify-demo
$ cd restify-demo
$ npm install restify
```

###Hello World

```javascript app.js
var restify = require('restify');

function respond(req, res, next) {
  res.send('hello ' + req.params.name);
}

var server = restify.createServer();
server.get('/hello/:name', respond);

server.listen(8080, function() {
  console.log('%s listening at %s', server.name, server.url);
});
```

保存文件并执行

```javascript
$ node app.js
restify listening at http://0.0.0.0:8080
```
通过curl命令测试，服务器返回HTTP状态码`200`和一个`application/json`类型的内容(事实上这里不是)

```javascript
$ curl -is http://localhost:8080/hello/roy
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Accept, Accept-Version, Content-Length, Content-MD5, Content-Type, Date, X-Api-Version
Access-Control-Expose-Headers: X-Api-Version, X-Request-Id, X-Response-Time
Server: restify
X-Request-Id: ec3e9e55-7083-4e12-93ee-9c24a0c5c398
Access-Control-Allow-Methods: GET
Connection: close
Content-Length: 5
Content-MD5: 65tW24XBT3r1jLUpw0o0Fw==
Content-Type: application/json
Date: Sun, 19 Feb 2012 16:14:13 GMT
X-Response-Time: 3

"hello roy"
```

###特性
接触到`restify`是因为内部系统需要一个手机归属地的查询接口，基于`nodejs`和`redis`的高性能实现，决定快速实现一个，当时也很自然地选择了`express`作为web层。但`express`始终是一个Web MVC框架，它包含了一些我不太需要的功能实现（i.e., templating/rendering）。而我只是需要一个更纯粹的API接口而已--接收请求然后返回一个`JSON`结果集。

后来从推上找到了`restify`时，我就觉得这才是我想要的：

1. 路由

	* 基本等同于`express`/`sinatra`的路由方式
	* 路由正则表达式支持
	* 版本化支持

2. 自动选择内容格式进行响应，并可以自定义格式
3. 内置REST风格的错误处理机制，重载了400和409来解析应答，得到一个包含`code`和`message`的JSON对象。也可以子类化`restify.RestError`实现自己错误代码和信息。
	
	已实现的RestError:
	* RestError
	* BadDigestError
	* BadMethodError
	* InternalErrorError
	* InvalidArgumentError
	* InvalidContentError
	* InvalidCredentialsError
	* InvalidHeaderError
	* InvalidVersionError
	* MissingParameterError
	* NotAuthorizedError
	* RequestExpiredError
	* RequestThrottledError
	* ResourceNotFoundError
	* WrongAcceptError

4. 插件机制

5. DTrace支持，这个绝对是调式、诊断和性能测量利器

虽然现在`restify`还是初生婴儿，我还是把上面提到的手机归属地查询接口改用了`restify`来实现了一个[moquery](https://github.com/roymax/moquery)。

正如官方描述一样
	
	restify is a smallish framework, similar to `express` for building REST APIs. 

