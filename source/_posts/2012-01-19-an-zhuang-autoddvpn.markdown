---
layout: post
title: "安装autoddvpn"
date: 2012-01-19 16:27
comments: true
published: true
categories: [效率]

---

在家已经成功部署autoddvpn近两个月，所以在家里上网的感觉特好（偶尔拉风一下），特记录下来为大家做一次明灯。

###动机

在用dd-wrt方案前，我用的是ssh tunnel再加上本地autopac，基本已经是一个无敌的解决方案了，也特靠谱的。但家里设备一多，你得想尽办法为每台设备折腾一下，找解决方案。折腾的这些东西都是很折腾人的。与其这样，何不在路由上做功夫，一劳永逸呢。

###需要环境
* 有一台可刷DD-WRT的设备 ( [你的路由支持吗？](http://dd-wrt.com/site/support/router-database) )
* 一个PPTP或OpenVPN 帐号
* ADSL上网环境

###准备
#####关于DD-WRT路由器
家里正好有两台设备，一台是D-Link DIR-605 D1 ,通过查询是wip状态:
	work in progress, router support is in the works, but please don't ask how long it takes, we cannot give you a schedule in the most cases.

另一台为Netgear WGR614 v9，非常悲剧也是wip。正要放弃准备购买新设备时得知，DIR-605可以刷DIR-615 D2，经过两个晚上的几个小时尝试最终也刷成功，但最后我还是买了一台路由器，因为DIR-605无论如何也支持不了OpenVPN。

	如果设备支持OpenVPN，在下载页会列出标有vpn字样的下载链接

我为什么需要OpenVPN？因为我拥有一个Astrill的VPN帐号[推广链接][1]，就因为这个家伙介绍支持DD-WRT我才知晓可以这样玩（*Astrill VPN本身是不支持多设备同时登录，通过DD-WRT可允许7台设备同时使用*）。当时我还不知道有autoddvpn这个东西，只能按Astrill的安装介绍来寻找一台支持OpenVPN版本的DD-WRT路由器了。而[Astrill推荐](https://www.astrill.com/knowledge-base/52/What-Router-for-OpenVPN.html)了**Asus RT-N16**和**Linksys WRT 160N**，刚好有朋友从香港回来就托运了一台，这样又多花费550人左右。所以我最后用的设备是RT-N16。

	事实上，你不需要OpenVPN，用PPTP也是可以的，只要按照autoddvpn的文档写好你的脚本。通过Astrill的安装界面只是图个方便而已。这里我用“可能”来描述是因为我没有进行测试。只不过到后来我也放弃了Astrill的内置安装模式。

#####关于PPTP或OpenVPN帐号
刚才我说了，我的解决方案是[Astrill][1]，大概一年70刀，最近又涨价了。而且现在竟然将DD-WRT单独出来做增值收费, 需要另付 $1/月。

如果你有root权限的主机，自建一个OpenVPN Server也是不错的选择。反正我就没有时间折腾，留给后来人做。

而淘宝上也有很多廉价VPN，总会有你合适的吧。想不花点钱还是不太靠谱吧？

顺便说说Astrill的优势在于服务器节点多，虽然很多速度也不怎么样，但香港节点和美国部分节点都非常的快，而且都不限流量。试过有一次通过New York节点下载XCode 4 ，是**1MB/秒**的速度。

#####ADSL
中国电信的8M ADSL

###安装调试
#####刷新固件
硬件准备好后就可以开始刷设备了，从dd-wrt.com下载合适的rom文件到本地。针对我的RT-N16，我一共下载了三个文件

* dd-wrt.v24-14896_NEWD-2_K2.6_mini_RT-N16.trx
* dd-wrt.v24-14896_NEWD-2_K2.6_mini.bin
* dd-wrt.v24-14896_NEWD-2_K2.6_openvpn.bin

用网线接上路由器，访问路由器设置网，找到通过本地上传方式**更新固件**的位置，必须先刷第一个文件，即扩展名为.trx的那个。刷新成功后，再刷其它，都非常简单就不表了。可以看看[这里](http://www.dd-wrt.com/wiki/index.php/Asus_RT-N16)

#####设置路由
如果使用的是Astrill服务，非常简单。连上网络后通过Astrill的客户端调出安装界面按提示执行就可以了。安装成功后在DD-WRT的路由设置页面进行设置，而且有专门针对GFW的设置，其实原理跟autoddvpn差不多，通过判断IP是否在墙内选择链路。

不过最后，我还是选择了使用autoddvpn介绍的最好模式graceMode。通过Astrill的傻瓜方式虽然可行，但：

* 连接服务不知为何非常的慢。
* 访问国内网站cdn cache会失效
* 依赖VPN的稳定性，试过几次VPN掉了连国内网站都访问不到

而autoddvpn是反向处理，使用的是本地DNS，对于部分指定的域名才走VPN链路。所以就算VPN不稳定的情况下，至少能保障国内的网站可以访问。

至于如何设置autoddvpn这里不说了，我认为官方的文档已经足够详细。如果你也像我一样使用Astrill VPN服务的话，通过ssh登录路由器，准备你的OpenVPN设置。

openvpn.conf文件需要用到的证书可以通过登录as trill的会员中心*VPN SERVICES -  OpenVPN certificates* 下载。*Server List*则可以获取当前的服务器状态和IP，选择几个最快的节点吧，我使用的是HK1和NY1。

###使用感受
两个月来，在家上网的体验是最好的，唯一不足是最近路由器竟然连接不上，会自动死掉也不会重启，重启后很快又正常。也不知道是否因为刷了DD-WRT而不稳定，我听朋友说他家的官ROM还是非常稳定的，有空时我想重刷一次看看效果。

这样作在家里用倒不错，但出门后就有点麻烦了。手机还好，需要那个情况不好，而且twitter也可以用API Proxy解决。所以我的Mac还是装用iSSH和PAC以备不时之需。其实我想过段日子每天出街自备一个TP—LINK TL-WR800N，连上网络后再接家里的VPN Server端口再访问出去……但多绕啊！再看看吧，不一定弄了。

最后感觉变成Astrill的推广文了，不写了…其实我觉得如果有Linode的东京节点，自建个OpenVPN Server应该也是不错的选择。

如有问题欢迎跟我交流联系.


[1]: http://xcarshop.com "Astrill推广链接"