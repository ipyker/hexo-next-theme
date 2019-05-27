layout: post
title: CentOS7 内核参数详解
author: Pyker
categories: Linux
img: /images/bar/kernel.jpg
tags:
  - Linux
top: false
cover: false
date: 2017-11-21 23:47:00
---
### sysctl.conf 配置参数详解
所谓Linux服务器内核参数优化，主要是指在Linux系统中针对业务服务应用而进行的系统内核参数调整，优化并无一定的标准。下面以生产环境下Linux常见的内核优化为例进行讲解，仅供参考。
```
$ cat /etc/sysctl.conf

#打开的文件句柄的数量
fs.file-max = 265535

#关闭ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

# 避免放大攻击
net.ipv4.icmp_echo_ignore_broadcasts = 1

# 开启恶意icmp错误消息保护
net.ipv4.icmp_ignore_bogus_error_responses = 1

#关闭路由转发
net.ipv4.ip_forward = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

#开启反向路径过滤
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

#处理无源路由的包
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0

#关闭sysrq功能
kernel.sysrq = 0

#core文件名中添加pid作为扩展名
kernel.core_uses_pid = 1

# 开启SYN洪水攻击保护
net.ipv4.tcp_syncookies = 1

#修改消息队列长度
kernel.msgmnb = 65536
kernel.msgmax = 65536

#设置最大内存共享段大小bytes
kernel.shmmax = 68719476736
kernel.shmall = 4294967296

#timewait的数量，默认180000
net.ipv4.tcp_max_tw_buckets = 6000
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_rmem = 4096        87380   4194304
net.ipv4.tcp_wmem = 4096        16384   4194304
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216

#每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目
net.core.netdev_max_backlog = 262144

#限制仅仅是为了防止简单的DoS 攻击
net.ipv4.tcp_max_orphans = 3276800

#未收到客户端确认信息的连接请求的最大值
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_timestamps = 0

#内核放弃建立连接之前发送SYNACK 包的数量
net.ipv4.tcp_synack_retries = 1

#内核放弃建立连接之前发送SYN 包的数量
net.ipv4.tcp_syn_retries = 1

#启用timewait 快速回收
net.ipv4.tcp_tw_recycle = 1

#开启重用。允许将TIME-WAIT sockets 重新用于新的TCP 连接
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_mem = 94500000 915000000 927000000
net.ipv4.tcp_fin_timeout = 1

#当keepalive 起用的时候，TCP 发送keepalive 消息的频度。缺省是2 小时
net.ipv4.tcp_keepalive_time = 30

#允许系统打开的端口范围
net.ipv4.ip_local_port_range = 1024    65000

#修改防火墙表大小，默认65536
net.netfilter.nf_conntrack_max=655350
net.netfilter.nf_conntrack_tcp_timeout_established=1200

# 确保无人能修改路由表
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.default.secure_redirects = 0
```
>`net.ipv4.tcp_tw_recycle = 1` 启用TIME-WAIT状态sockets的快速回收，这个选项不推荐启用。在NAT(Network Address Translation)网络下，会导致大量的TCP连接建立错误。
>
>我们在一些高并发的 WebServer上，为了端口能够快速回收，常常会打开了 tcp_tw_reccycle 。在关闭 tcp_tw_reccycle 的时候，kernel 是不会检查对端机器的包的时间戳的；而打开了 tcp_tw_reccycle 了，就会检查时间戳，很不幸网络发来的包的时间戳是乱跳的，所以我方的就把带了“倒退”的时间戳的包当作是“recycle的tw连接的重传数据，不是新的请求”，于是丢掉不回包，造成大量丢包。


>当运行`sysctl -p`命令时报error: 'net.ipv4.ip_conntrack_max' is an unknown key 错时,
通过以下命令修正：
$ modprobe ip_conntrack
$ echo "modprobe ip_conntrack" >> /etc/rc.local


### 一套生产环境使用过的内核参数
* sysctl.conf文件

```
fs.file-max=65535
net.ipv4.ip_forward = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0

net.ipv4.tcp_max_tw_buckets = 6000
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_tw_recycle = 0

net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_syncookies = 1

kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296

net.ipv4.tcp_max_syn_backlog = 262144
net.core.netdev_max_backlog = 262144
net.core.somaxconn = 262144
net.ipv4.tcp_max_orphans = 262144

net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_fin_timeout = 1
net.ipv4.tcp_keepalive_time = 30

net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_wmem = 4096 16384 4194304
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_mem = 94500000 915000000 927000000
net.nf_conntrack_max = 6553500
#redis
vm.dirty_ratio=10
vm.dirty_background_ratio=5
```

* 禁用SELINUX

```
$ vi /etc/sysconfig/selinux  
设置为disabled
```
* 同步时间

```
$ crontal -l
*/20 * * * * /usr/sbin/ntpdate pool.ntp.org > /dev/null 2>&1
```

* 文件描述符数量修改`/etc/security/limits.conf`文件，在文件末尾添加

```
root soft nofile 65535
root hard nofile 65535
* soft nofile 65535
* hard nofile 65535
```
还需要修改`/etc/security/limits.d`下面的conf文件(会覆盖前面的配置信息)，我的是20-nproc.conf
```
* soft nproc 65535
* hard nproc 65535
```

* 禁用自带的Firewalld防火墙

```
systemctl stop firewalld.service 停止firewall
systemctl display firewalld.service 禁止firewall开机自启动
```