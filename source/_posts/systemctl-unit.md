layout: post
title: systemctl unit服务格式
author: Pyker
categories: Linux
img: /images/bar/systemctl.jpg
tags:
  - Linux
top: false
cover: false
date: 2017-11-16 17:10:00
---
systemctl是RHEL 7 的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体。可以使用它永久性或只在当前会话中启用/禁用服务。
## 服务权限
systemd有系统和用户区分；系统`/user/lib/systemd/system/`, 用户`/etc/lib/systemd/user/`一般系统管理员手工创建的单元文件建议存放在`/etc/systemd/system/`目录下面。或者`/usr/lib/systemd/system/`下面 ，然后可以通过`systemctl enable xxx.service`方式将该服务添加到`/etc/systemd/system/multi-user.target.wants/`目录下面设置为开机自启动。
## 格式介绍
systemctl的服务文件主要包含`[Unit]`、`[Service]`、`[Install]`三类。下面我们对这三类进行说明。
### [Unit]
该部分主要对服务进行说明。
  * `Description` : 服务的简单描述。
  * `Documentation` ： 服务文档地址说明。
  * `Before=xxx.service`：代表本服务在xxx.service启动之前启动。
  * `After=xxx.service`：代表本服务在xxx.service启动之后启动。
  * `Requires`： 本服务启动后，它需要的服务也会启动；而本服务需要的服务被停止了，本服务也停止了。
  * `Wants`： 推荐使用。本服务启动了，它需要的服务也会被启动；而本服务需要的服务被停止了，对本单元没有影响。
---
### [Service]
该部分的配置服务的启动、重启、停止命令全部要求使用绝对路径，使用相对路径则会报错。
  * `WorkingDirectory`：指定服务运行的工作目录。

  * `EnvironmentFile`：指定服务需要的配置文件，下文可以用`${}`方式引用文件中的值。

  * `Type=simple（默认值）`：systemd认为该服务将立即启动。服务进程不会fork。如果该服务要启动其他服务，不要使用此类型启动，除非该服务是socket激活型。

  * `Type=forking`：systemd认为当该服务进程fork，且父进程退出后服务启动成功。对于常规的守护进程（daemon），除非你确定此启动方式无法满足需求，使用此类型启动即可。**使用此启动类型应同时指定 PIDFile=**，以便systemd能够跟踪服务的主进程。

  * `Type=oneshot`：这一选项适用于只执行一项任务、随后立即退出的服务。可能需要**同时设置 RemainAfterExit=yes** 使得 systemd 在服务进程退出之后仍然认为服务处于激活状态。

  * `Type=notify`：与 Type=simple 相同，但约定服务会在就绪后向 systemd 发送一个信号。这一通知的实现由 libsystemd-daemon.so 提供。

  * `Type=dbus`：若以此方式启动，当指定的 BusName 出现在DBus系统总线上时，systemd认为服务就绪。

  * `Type=idle`: systemd会等待所有任务(Jobs)处理完成后，才开始执行idle类型的单元。除此之外，其他行为和Type=simple 类似。

  * `PIDFile`：指定pid文件路径

  * `ExecStart`：指定启动单元的命令或者脚本

  * `ExecStartPre和ExecStartPost`：指定在执行ExecStart之前或者之后执行用户自定义的脚本，**Type=oneshot**允许用户执行多个按顺序定义的脚本。

  * `ExecReload`：指定单元重载时执行的命令或者脚本。

  * `ExecStop`：指定单元停止时执行的命令或者脚本。

  * `PrivateTmp`：True表示给服务分配独立的临时空间

  * `LimitNOFILE`：设置文件描述符相关参数个数，或者设置为无穷 infinity

  * `LimitNPROC`：设置文件描述符相关参数个数，或者设置为无穷 infinity

  * `LimitCORE`：设置文件描述符相关参数个数，或者设置为无穷 infinity

  * `Restart`：这个选项如果被允许，服务重启的时候进程会退出，会通过systemctl命令执行清除并重启的操作。通常设置为**on-failure**。

  * `RemainAfterExit`：如果设置这个选择为真，服务会被认为是在激活状态，即使所以的进程已经退出，默认的值为假，这个选项只有在Type=oneshot时需要被配置。
---
### [Install]
定义如何安装这个配置文件，即怎样做到开机启动。
  * `Alias`：为单元提供一个空间分离的附加名字。
  * `RequiredBy`：单元被允许运行需要的一系列依赖单元，RequiredBy列表从Require获得依赖信息。
  * `WantBy`：单元被允许运行需要的弱依赖性单元，Wantby从Want列表获得依赖信息。
  * `Also`：指出和单元一起安装或者被协助的单元。
  * `DefaultInstance`：实例单元的限制，这个选项指定如果单元被允许运行默认的实例。
---
## mysql.service样例
```bash
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Service]
User=mysql
Group=mysql
Type=forking
PIDFile=/var/run/mysqld/mysqld.pid
TimeoutSec=0
PermissionsStartOnly=true
ExecStartPre=/usr/bin/mysqld_pre_systemd
ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS
EnvironmentFile=-/etc/sysconfig/mysql
LimitNOFILE = 5000
Restart=on-failure
RestartPreventExitStatus=1
PrivateTmp=false

[Install]
WantedBy=multi-user.target
```
## etcd.service样例
```bash
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf          #etcd配置文件路径
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/bin/etcd --name=\"${ETCD_NAME}\" --data-dir=\"${ETCD_DATA_DIR}\" --listen-client-urls=\"${ETCD_LISTEN_CLIENT_URLS}\""
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target    # 说明：其中WorkingDirectory为etcd数据库目录，需要在etcd**安装前创建**
```
