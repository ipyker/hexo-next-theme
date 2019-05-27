layout: post
title: Zabbix服务器监控之《配置Zabbix监控项》（四）
author: Pyker
categories: Zabbix
img: /images/bar/zabbix-04.jpg
tags:
  - zabbix
top: false
cover: false
date: 2018-09-12 20:10:00
---
通过[上一篇](https://www.ipyker.com/2018/09/11/zabbix-agent/) 我们已经进行了对zabbix的agent端安装，以及通过手动、自动发现、自动注册的方式将agent端监控到server端，也知道`应用集`、`监控项`、`触发器`是什么概念。那么本篇我们将实际配置zabbix的监控项。

## 监控项说明
在Zabbix中，我们要监控的某一个指标，被称为“监控项(item)”，比如监控磁盘的使用率，这就属于一个监控项，如果要获取到"监控项"的相关信息，我们则要执行一个命令，但是我们不能直接调用命令，而是通过一个"别名"去调用命令，这个"命令别名"在zabbix中被称为"键"(key)，所以，在zabbix中，如果我们想要获取到一个"监控项"的值，则需要有对应的"键"，通过"键"能够调用相应的命令，获取到对应的监控信息，而监控项的key、键值又可以为`带参数`和`不带参数`两种，下面我们将分别对其进行说明配置。 
* **不参数键值**
监控项键值中只有键名的键称为不带参数的键值，如`system.cpu.switches` 
* **带参数的键值**
监控项键值中有键名和参数的键值，如`vfs.fs.size[fs,<mode>]`，对于`vfs.fs.size[fs,<mode>]`这个键来说，`vfs.fs.size`就是键名，`[fs,<mode>]`就是这个键需要的参数。而`[fs,<mode>]`这两个参数中，`fs`是不可省参数，`mode`是可省参数。

## 配置监控项
监控项可以单独配置在主机中，使该监控项专属于该主机。也可以配置在相应模版中，然后主机对该模版进行链接，从而使该主机也具有该模版的所有监控项。 

<font color=#FF0000>在通常配置zabbix监控时，我们通常将监控项配置在模版中，这样方便调度和管理。(以下配置也采用监控项配置在模版里)</font>
>系统默认自带一些模版，这些模版有监控CPU的、内存的、http的等等，如果没有自己需要的也可以自定义模版，应用集、监控项、触发器等。

### 配置不带参数的监控项
点击`配置-模版-Template OS Linux` 找到Template OS Linux模版
![](/images/pic/zabbix/zabbix-config021.png)

点击Template OS Linux模版中的`监控项-创建监控项`，完成下图设置后`添加`
现在我们想要在Template OS Linux模版中创建CPU的上下文切换的监控项，那么我们可以在此界面进行如下配置。 
![](/images/pic/zabbix/zabbix-config022.png)
现在我们可以看到在Template OS Linux模版中已经有我们刚刚创建的名为Context switches per second监控项
![](/images/pic/zabbix/zabbix-config023.png)
此时我们让主机和该模版关联，使主机能使用该模版的监控项， 点击`配置-主机-模版`，把Template OS Linux模版链接到主机中
![](/images/pic/zabbix/zabbix-config024.png)
完成以上操作后，我们对主机进行cpu上下文监控也就配置完成，可以等待2分钟让zabbix进行数据采集后，在`监测-最新数据`通过过滤器过滤出CPU上下文切换的监控项最新数据
![](/images/pic/zabbix/zabbix-config025.png)
也可以点击旁边的图形，查看图形信息
![](/images/pic/zabbix/zabbix-config026.png)

### 配置带参数的监控项
点击Template OS Linux模版中的`监控项-创建监控项`，由于配置项已在上面说明，我们只阐述不同的地方
![](/images/pic/zabbix/zabbix-config027.png)
由于Template OS Linux模版在上面已经加入到nginx1主机了，所以我们这里就不做主机和模版关联了
![](/images/pic/zabbix/zabbix-config024.png)
最后等待几分钟在`监测-最新数据`通过过滤器过滤出磁盘使用率的监控项最新数据
![](/images/pic/zabbix/zabbix-config028.png)

>关于这个键到底怎么使用呢，类似和fs和mode这两个参数分别代表了什么呢，我们可以通过[官网帮助手册](https://www.zabbix.com/documentation/4.0/zh/manual/config/items/itemtypes/zabbix_agent)，查看这些"键"的含义与使用方法。