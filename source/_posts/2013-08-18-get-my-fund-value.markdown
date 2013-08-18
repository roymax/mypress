---
layout: post
title: "get my fund value"
date: 2013-08-18 20:17
comments: true
external-url: 
published: true
categories: 
---

回想2008年，头脑发热买了一批基金，而且是在暴跌的前夕。无需有太多的猜想也知道，必亏无疑的，直到今天每一只在未计息和通货的情况下也亏损30%左右。是的，无论基于什么价值投资也好，沉没成本也好我都好像应该卖了它，事实上是我一直也没有将它们卖出。每隔一段时间我都会看看他们的「收益」如何。

基金工具有很多，但没有一个我觉得称心的，我只是想偶尔看一下，不想录入太多不必要的表单。我只想录入基金编号，本金和份额，得到当前的价值即可。

以前，我是用一个Excel文件记录下每一只基金，忘记了调用一个什么函数动态从一个基金网站抓取当天的净值数据，后来这个文件丢失了，我只好每次查询每一只基金并输入净值数据。

这个周末，刚好看到一个便捷的[基金接口](http://www.juhe.cn/docs/api/id/25)后，我决定写一个小脚本完成我的小需求。

{% gist 6261332 data.yaml %}

{% gist 6261332 fund.rb %}

`ruby 1.9.3`测试通过，依赖`httparty`,`settingslogic`,`text-table`,`colorize`，运行前请确认已经安装以上gem，同时需要到[juhe.cn](http://www.juhe.cn)申请一个API KEY。

运行效果，看这个

{% img http://d.pr/i/gjbf+ %} 