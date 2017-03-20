---
title: 路由表
date: 2016-06-11 18:01:50
tags: route
categories: 网络
---

# 路由表

路由表应用在tcp/ip中的ip层协议中，当系统中的ip数据包组装完毕后，就要决定它的发送方向了，这个ip包有可能是发送给本地的，比如127.0.0.1上的某个服务器，例如本机的mysql监听127.0.0.1的3306端口,也有可能这个ip包是发送给局域网某台机器上面的，也许是发送给百度的，例如在浏览器中输入www.baidu.com后按下回车。在这么多选择中，如何能让这个ip包正确的被发送出去，这个就要使用路由表来做这个事情。
<!--more-->
![windows 路由表](/site_files/20160625-1751.png)
在windows 7 操作系统上，在 cmd窗口里面输入 route print -4 就会出现如上的结果，这张表就是windows的ipv4的路由表。
在上图的这个路由表中，在”活动路由:”下面那一行中，有网络目标、网络掩码、网关、接口和跃点数。

这这个表中，网络目标是最重要的参数，每当有ip包要发送时，首先就是要看ip包中的destination address，然后在网络目标这一列中寻找是否有一样的地址，
* 比如IP包中的destination address是127.0.0.1的话，那么这个数据包就会发到接口地址为127.0.0.1上面了，其实就是发送到本地网络上面了；
* 那么如果地址是172.16.xx.xx,那么这个地址就是符合上表中的第五条记录了，那么此数据包就会被发送172.16.1.107网卡上，然后接下来这个数据包就会发送到局域网中，被那台局域网机器收走[此过程被极度简化，有时间再补充]；
* 如果一个ip包在此路由表中一个也找不到，那么就会走0.0.0.0这个规则，这个0.0.0.0的规则原则上就是如果在这台机器的路由表中如果没有找到符合这个ip包的发送规则时，就把这个数据包转发给这台机器所处的网络的路由器或者交换机上，也就是说0.0.0.0就是在没有寻找到目的地址时就把数据包丢给上一级网络，让上一级网络去处理，在不停的经历这个过程后，这个ip包会最终被发送给对应的机器中，在上表中，0.0.0.0这个规则是将数据包转发到这个局域网中的路由器上了，然后如果路由器发现这个目的地址在它的路由表中也没有相关表项可以处理，那么路由器也会把这个数据包丢给它的上一级网络去处理，知道发送给对应的机器上为止，也有一种情况就是目的地址根本不存在，那么这个最终就会发送失败了。

实际上在路由寻找时，网络目标和网络掩码还有跃点数等参数都是配合使用的，并不只有网络目标在起作用，网络掩码有个规则是” 路由掩码最长匹配原则”,如果网络目标都是0.0.0.0，一个路由条目中的掩码是 0.0.0.0,另一个是128.0.0.0,那么在进行路由匹配时，就会走网络掩码为128.0.0.0那一条路由规则。而跃点数的概念可以参考这个 “跃点数表明使用路由到达目标位置的相对成本。常用指标为跃点，或到达目标位置所通过的路由器数目。如果有多个相同目标位置的路由，跃点数最低的路由为最佳路由。”

上面所讲的规则，可以说是路由策略，在linux系统中，除了普通的路由表，即路由策略外，还有一种策略路由，策略路由应用的方面就更广阔了。策略路由可以说是一种高级的静态路由，但是又与静态路由不同。普通的静态路由（也包括动态路由）是按照数据包的目的地址来进行路由，而策略路由是按照数据包的源地址来进行路由。在同一台路由器上如果配置了策略、静态、动态三种路由，路由器接口首先对入站的数据包源地址进行判>断有没有匹配在此接口上配置的策略路由的数据流，如果有，则按照策略路由的配置转发数据包。如果没有，则按普通数据包情况路由。具体是静态>路由协议优先还是动态路由协议优先（去往同一个目的地址根据路由协议的不同有多条路径），要看你在此路由器上定义的管理具体大小，管理距离>越小则此种路由协议的可信度越高，则优先选用该种路由协议。而管理距离的默认值又根据各厂家路由器的不同而不同。策略路由让linux有了256张路由表，可以定义优先级，可以根据ip包中的源地址，将指定的数据包送到对应的路由表中，也可以根据目的地址将ip包做负载均衡等等。

这篇文章表述了networking为何物[Linux Networking-concepts HOWTO](http://www.netfilter.org/documentation/HOWTO/networking-concepts-HOWTO.html)非常简短明了的描述了网络概念,而[Linux Networking-HOWTO](http://www.tldp.org/HOWTO/NET3-4-HOWTO.html)描述了linux 网络的基础情况.

参考地址
[路由掩码最长匹配原则和 IP Classless](http://zjskobe.blog.51cto.com/2772091/724307)
[Linux Advanced Routing & Traffic Control HOWTO](http://lartc.org/howto/)
