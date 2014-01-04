---
layout: post
title: "Happy new year to you!"
date: 2014-01-04 15:03
comments: true
external-url: 
published: true
categories: ['ruby']
---

```
$ pry

2.0.0 (main):0 > 'Happy new year to you!'.each_byte.inject(:+)
=> 2014
```

偶尔在 twitter 上看到这个，惊喜之余翻出`inject`探个究竟

文档给出这样的例子

```
(5..10).inject(:+)
=> 45
``` 

`inject`可以解释作**注入**，它是`Enumerable`其中的一个实例方法，实现了`Enumerable`模块的类都可以使用它的实例方法，`Array`类当然实现了它。所以`inject`的作用就是在每个数组元素之间进行你想做的处理。**Ruby** 的API 文档给出了很直观的例子。

回到开头那行代码就很直白了，将字符串转换成字节码然后相加，最终得到**2014**，令人愉悦的是那个字符串是*Happy new year to you!*。

尝试用Java来写一下

```
	public static void main(String[] args) {
		String str = "Happy new year to you!";

		int sum  = 0;
		for(byte b: str.getBytes()) {
			sum += b;
		}
		
		System.out.println(sum);
	}
```

