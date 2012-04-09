---
layout: post
title: "mac osx自动添加VPN路由"
date: 2012-04-09 17:51
comments: true
published: true
categories: [shell, osx]

---

最近需要频繁使用VPN拨号，但由于网段不一样，每次连上后都需要手工添加路由，在Mac下同样可以通过ip-up和ip-down自动完成路由的添加。

{% gist 2342505 %}

非常简单的代码，其中第7行的${5:-} 对应远程IP地址，另还有以下参数可以使用
	
	$1 the interface name used by pppd (e.g. ppp3)
	$2 the tty device name
	$3 the tty device speed
	$4 the local IP address for the interface
	$5 the remote IP address
	$6 the parameter specified by the 'ipparam' option to pppd
	
最后，必须赋执行权限

```
$ sudo chmod a+x /etc/ip-up

#查看是否添加成功
$ netstat -nr
```

