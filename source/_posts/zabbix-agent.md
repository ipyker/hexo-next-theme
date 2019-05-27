layout: post
title: Zabbix服务器监控之《安装Zabbix Agent》（三）
author: Pyker
categories: Zabbix
img: /images/bar/zabbix-03.jpg
tags:
  - zabbix
top: false
cover: false
date: 2018-09-11 14:51:30
---
[上一篇](https://www.ipyker.com/2018/09/11/zabbix-server/) 我们已经进行了对zabbix的server端安装，本篇我们将进行对Zabbix agent端安装配置！

通过[第一篇](https://www.ipyker.com/2018/09/11/zabbix-summarize/) 我们已经知道`server-agent` 都是部署到被监控主机上，由agent采集数据，报告给zabbix-server端进行监控的。
## 安装Zabbix代理端
>由于zabbix-server用的是4.2版，那么我们agent也用4.2版。

### yum安装zabbix-agent
```bash
#yum安装zabbix源
$ rpm -Uvh https://repo.zabbix.com/zabbix/4.2/rhel/7/x86_64/zabbix-release-4.2-1.el7.noarch.rpm

# 安装zabbix-agent代理端
# zabbix_sender` 为测试是否能够向server端发送数据的工具
$ yum install -y zabbix-agent zabbix-sender
```
### zabbix代理端配置
```bash
# 修改/etc/zabbix/zabbix_agentd.conf配置文件
# 设置被动模式下zabbix-server的IP地址
Server=192.168.20.210

# 设置主动模式下zabbix-server的IP地址
ServerActive=192.168.20.210

# 设置本机zabbix-agent主机名称
Hostname=web server1
```
### 启动zabbix代理端
```bash
# 启动zabbix-agent
$ systemctl start zabbix-agent

# 开机自启动zabbix-agent`
$ systemctl enable zabbix-agent
```
>默认情况下，zabbix-agent不能使用root用户运行的，如果非要以root用户运行，可以在`/etc/zabbix/zabbix_agentd.conf` 配置文件中设置`AllowRoot=1` 

**此时查看zabbix-agent进程和端口是否正常：**
```bash
# 查看端口是否正常
$ netstat -ptln
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      11734/zabbix_agentd 

# 查看进程是否正常
$ ps -ef | grep zabbix_agent
zabbix    11734      1  0 4月13 ?       00:00:00 /usr/sbin/zabbix_agentd -c /etc/zabbix/zabbix_agentd.conf
zabbix    11735  11734  0 4月13 ?       00:00:00 /usr/sbin/zabbix_agentd: collector [idle 1 sec]
zabbix    11736  11734  0 4月13 ?       00:00:00 /usr/sbin/zabbix_agentd: listener #1 [waiting for connection]
zabbix    11737  11734  0 4月13 ?       00:00:00 /usr/sbin/zabbix_agentd: listener #2 [waiting for connection]
zabbix    11738  11734  0 4月13 ?       00:00:00 /usr/sbin/zabbix_agentd: listener #3 [waiting for connection]
zabbix    11739  11734  0 4月13 ?       00:00:00 /usr/sbin/zabbix_agentd: active checks #1 [idle 1 sec]
root      12105  11549  0 00:23 pts/0    00:00:00 grep --color=auto zabbix_agent
```

至此`zabbix-agent` 代理端安装完毕。
## 监控zabbix-agent代理端
zabbix-server监控zabbix-agent有两种方式：
* 手动添加对应的zabbix-agent客户端
* 自动发现和自动注册

>手动添加zabbix-agent适用于少量的被监控主机，而自动发现主要是通过发现网络中的主机，并自动把主机添加到监控中，并关联特定的模板，实现自动监控，适合有大量被监控主机，减少频繁手动添加主机的麻烦操作。

### 手动添加zabbix-agent客户端
浏览器输入`http://192.168.20.210/zabbix` ，访问zabbix服务，点击`配置-主机-创建主机` 
![](/images/pic/zabbix/zabbix-config010.png)
点击`模版` ，选择主机属于哪个模版 （模版里面包含需要监控的条目）
![](/images/pic/zabbix/zabbix-config011.png)
点击`添加`
![](/images/pic/zabbix/zabbix-config012.png)
* `应用集`：  表示模版中监控业务的一类型。（如：CPU应用集属于Template OS Linx模版）
* `监控项`：  表示某一应用集中监控的某一项。（如：CPU负载、CPU使用率都属于CPU应用集中的监控项）
* `触发器`：  表示某一监控项达到某自定义的阀值时，该出现的状态。（如：当CPU使用率达到70%设置他的状态为警告，80%为严重，这就属于触发器）
* `图 形`：   表示哪些监控项进行了图形显示。（如：将CPU使用率用图形显示一段时间里的曲线变化）
* `自动发现`：表示哪些监控项是可以通过自动发现进行监控。（如：自动发现主机网络接口流量）
* `web监测`:  表示zabbix把web某页面也监控起来，第一时间得知web崩溃信息并做相应处理。（如：监控http某一页面，如果状态码不是200，通过触发器返回严重告警）

至此手动添加zabbix-agent客户端步骤完成，可以在`监控-图形`中查看对监控项设置了图形的状态了。
![](/images/pic/zabbix/zabbix-config013.png)

### 自动发现
当监控主机不断增多，有的时候需要添加一批机器，特别是刚用zabbix的运维人员需要将公司的所有服务器添加到zabbix，如果使用传统办法去单个添加设备、分组、项目、图像…..结果可想而知。鉴于这个问题我们可以好好利用下Zabbix的一个发现(Discovery)模块，进而来实现自动刚发现主机、自动将主机添加到主机组、自动加载模板、自动创建项目（item）、自动创建图像，下面我们来看看这个模块如何使用。
>自动发现由服务端主动发起，Zabbix Server开启发现进程，定时扫描局域网中IP服务器、设备。

**自动发现过程需要分为两个步骤：**
* 通过网络扫描制定的服务，如：Zabbix Agent是否可以访问system.uname指标
* 发现主机之后需要执行添加的动作，这个过程由动作（Action）完成

点击`配置-自动发现-创建发现规则`填入名称、需发现服务器、设备的IP范围、更新间隔、检查项（ssh和zabbix客户端）、设备唯一性准则，最后勾选已启用、点击添加。进行自动发现规则的创建
![](/images/pic/zabbix/zabbix-config014.png)

在点击`配置-动作-事件源-自动发现-创建动作` 进行主机自动加入主机组并关联模板
![](/images/pic/zabbix/zabbix-config015.png)
点击`操作` 对该动作关联到模版的操作
![](/images/pic/zabbix/zabbix-config016.png)
`添加到主机`：将发现到的主机添加到主机
`添加到主机群组`：选择要添加到的主机组 
`链接到模版`：链接到模板、选择相应的模板 

点击`添加`完成动作规则的创建，至此发现主机、添加主机并将主机添加到主机组、链接模板全部完毕。

此时可以在`监测-自动发现`中看到已经被自动发现规则监测到的内网服务器！也可以在`配置-主机群组-Discovered hosts`中看到已经被自动发现的主机。也可以在`监测-图形`中查看自动发现的主机监控项的图形。
![](/images/pic/zabbix/zabbix-config017.png)
### 自动注册
当主机分布在不同的城市，比如不同的云环境中时，使用主动发现就不好处理了，使用自动注册的方式非常适合在云环境中的部署。
>由客户端主动发起，客户端必须安装并启动Agentd，否则无法被自动注册添加至主机列表。

点击`配置-动作-事件源-自动注册-创建动作` 进行主机自动加入主机组并关联模板
![](/images/pic/zabbix/zabbix-config018.png)
点击`操作`选择具体的操作类型：添加主机、添加到主机群组、与模板关联的操作，最后点击添加。

![](/images/pic/zabbix/zabbix-config019.png)

在`配置-主机`中查看注册的设备信息
![](/images/pic/zabbix/zabbix-config020.png)