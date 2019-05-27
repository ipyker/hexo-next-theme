layout: post
title: Zabbix服务器监控之《初识zabbix概念》（一）
author: Pyker
categories: Zabbix
img: /images/bar/zabbix-01.jpg
tags:
  - zabbix
top: false
cover: false
date: 2018-09-11 12:27:00
---
## 为什么要用Zabbix
对于运维人员来说，监控是非常重要的，因为如果想要保证线上业务整体能够稳定运行，那么我们则需要实时关注与其相关的各项指标是否正常，而一个业务系统的背后，往往存在着很多的服务器、网络设备等硬件资源，如果我们想要能够更加方便的、集中的监控他们，我们则需要依靠一些外部的工具，而zabbix就是一个被广泛使用的，可以实现集中监控管理的应用程序。

## Zabbix支持那些通讯方式
* `agent` ：通过专用的代理程序进行监控，与常见的master/agent模型类似,如果被监控对象支持对应的agent，推荐首选这种方式。
* `ssh/telnet` ：通过远程控制协议进行通讯，比如ssh或者telnet。
* `SNMP` ：通过SNMP协议与被监控对象进行通讯，SNMP协议的全称为Simple Network Management Protocol ,被译为 "简单网络管理协议"，通常来说，我们无法在路由器、交换机这种硬件上安装agent，但是这些硬件往往都支持SNMP协议，SNMP是一种比较久远的、通行的协议，大部分网络设备都支持这种协议，其实SNMP协议的工作方式也可以理解为master/agent的工作方式，只不过是在这些设备中内置了SNMP的agent而已，所以，大部分网络设备都支持这种协议。
* `IPMI` ：通过IPMI接口进行监控，我们可以通过标准的IPMI硬件接口，监控被监控对象的物理特征，比如电压，温度，风扇状态，电源状态等。
* `JMX` ：通过JMX进行监控，JMX（Java Management Extensions，即Java管理扩展），监控JVM虚拟机时，使用这种方法也是非常不错的选择。

## Zabbix监控流程
一般情况下，我们将zabbix agent部署到被监控主机上，由agent采集数据，报告给负责监控的中心主机，中心主机也就是master/agent模型中的master，负责监控的中心主机被称为zabbix server，zabbix server将从agent端接收到的信息存储于zabbix的数据库中，我们把zabbix的数据库端称为zabbix database， 如果管理员需要查看各种监控信息，则需要zabbix的GUI，zabbix的GUI是一种Web GUI，我们称之为zabbix web，zabbix web是使用php编写的，所以，如果想要使用zabbix web展示相关监控信息，需要依赖LAMP环境，不管是zabbix server ，或是zabbix web，他们都需要连接到zabbix database获取相关数据，这样说可能不容易理解，对比下图理解上述概念，就容易许多。
![](/images/pic/zabbix/zabbix1.png)

当监控规模变得庞大时，我们可能有成千上万台设备需要监控，这时我们是否需要部署多套zabbix系统进行监控呢？如果部署多套zabbix监控系统，那么监控压力将会被分摊，但是，这些监控的对象将会被尽量平均的分配到不同的监控系统中，这个时候，我们就无法通过统一的监控入口，去监控这些对象了，虽然分摊了监控压力，但是也增加了监控工作的复杂度，那么，我们到底该不该建立多套zabbix监控系统从而分摊巨大的监控压力呢？其实，zabbix天生就有处理这种问题的能力，因为zabbix支持分布式监控，我们可以把成千上万台的被监控对象分成不同的区域，每个区域中设置一台代理主机，区域内的每个被监控对象的信息被agent采集，提交给代理主机，在这个区域内，代理主机的作用就好比zabbix server，我们称这些代理主机为zabbix proxy，zabbix proxy再将收集到的信息统一提交给真正的zabbix server处理，这样，zabbix proxy分摊了zabbix server的压力，同时，我们还能够通过统一的监控入口，监控所有的对象，当监控规模庞大到需要使用zabbix proxy时，zabbix的架构如下图，我们可以对比下图，理解上述描述。
![](/images/pic/zabbix/zabbix2.png)

## 总结
### zabbix核心组件
* `zabbix agent` ：部署在被监控主机上，负责被监控主机的数据，并将数据发送给zabbix server。
* `zabbix server` ：负责接收agent发送的报告信息，并且负责组织配置信息、统计信息、操作数据等。
* `zabbix database` ：用于存储所有zabbix的配置信息、监控数据的数据库。
* `zabbix proxy` ：可选组件，用于分布式监控环境中，zabbix proxy代表server端，完成局部区域内的信息收集，最终统一发往server端。

### zabbix工作模式
我们知道，agent端会将采集完的数据主动发送给server端，这种模式我们称之为主动模式，即对于agent端来说是主动的。
其实，agent端也可以不主动发送数据，而是等待server过来拉取数据，这种模式我们称之为被动模式。
但是，不管是主动模式还是被动模式，都是对于agent端来说的，而且，主动模式与被动模式可以同时存在，并不冲突。
管理员可以在agent端使用一个名为`zabbix_sender` 的工具，测试是否能够向server端发送数据。
管理员也可以在server端使用一个名为`zabbix_get` 的工具，测试是否能够从agent端拉取数据。