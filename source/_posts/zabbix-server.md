layout: post
title: Zabbix服务器监控之《安装Zabbix Server》（二）
author: Pyker
categories: Zabbix
img: /images/bar/zabbix-02.jpg
tags:
  - zabbix
top: false
cover: false
date: 2018-09-11 13:47:00
---
[上一篇](https://www.ipyker.com/2018/09/11/zabbix-summarize/) 我们已经讲了zabbix的常用组件和工作模式，本篇我们将进行对Zabbix Server端安装配置！
## 搭建环境 
### 系统信息 
系统 | 版本 | IP | 关系
---- | ---- | -- | ----
centos | 7.5 | 192.168.20.210 | 服务端
centos | 7.5 | 192.168.20.211 |  代理端

### 环境配置 
* 设置主机名，重启生效

```bash
# server端
$ echo "zabbix-server" > /etc/hostname
 
# agent端
$ echo "zabbix-agent" > /etc/hostname
```
* 关闭SELinux和防火墙检查

```bash
$ sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
$ systemctl stop firewalld.service
$ systemctl disable firewalld.service
```
## 安装Zabbix服务端 
>我们这里安装的zabbix版本为4.2版本

### yum安装zabbix-server
```bash
#yum安装zabbix源
$ rpm -Uvh https://repo.zabbix.com/zabbix/4.2/rhel/7/x86_64/zabbix-release-4.2-1.el7.noarch.rpm

#安装zabbix服务端
#zabbix-get为测试是否能够从agent端拉取数据的工具
$ yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-agent zabbix-get
```
### 安装mysql数据库
>如果有现有的数据库环境，请跳过安装数据库环节，直接从`创建zabbix数据库` 开始。

```bash
# 在线yum安装mysql5.7
$ wget -c https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
$ rpm -ivh mysql80-community-release-el7-1.noarch.rpm
$ yum -y install yum-utils
$ yum-config-manager --disable mysql80-community
$ yum-config-manager --enable mysql57-community
$ yum install mysql-community-server -y

# 启动mysql
$ systemctl start mysqld

# 开机启动
$ systemctl enable mysqld
```
修改root密码
```bash
# 查看mysql临时密码
$ grep 'temporary password' /var/log/mysqld.log

# 使用mysql临时登录，修改root密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Ala@2018';
```
### 创建zabbix数据库
创建zabbix用户和库
```bash
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by "Ala@2018";
```
### 导入zabbix数据库
在shell命令行执行导入zabbix数据
```bash
$ zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p'Ala@2018' zabbix
```
### zabbix服务端配置
```bash
# 配置zabbix连接的数据库地址、数据库名以及数据库用户
DBHost=localhost
DBName=zabbix
DBUser=zabbix

# 修改`/etc/zabbix/zabbix_server.conf` 文件，修改mysql连接密码
DBPassword=Ala@2018

# 添加上海区
$ sed -i.ori '19a php_value date.timezone  Asia/Shanghai' /etc/httpd/conf.d/zabbix.conf

# 解决图形列表下中文乱码
$ yum -y install wqy-microhei-fonts
$ mv /usr/share/fonts/dejavu/DejaVuSans.ttf /usr/share/fonts/dejavu/DejaVuSans.ttf.bak
$ cp -f /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf
```
### 启动zabbix服务端并配置
```bash
# 启动 zabbix-server和httpd服务
$ systemctl start zabbix-server httpd

# 开机启动
$ systemctl enable zabbix-server httpd
```
浏览器输入`http://192.168.20.210/zabbix` ，访问zabbix，如下图
![](/images/pic/zabbix/zabbix-config001.png)
接下来点击 Next setup
![](/images/pic/zabbix/zabbix-config002.png)
从上图可以看到zabbix相关组件配置，继续点击 Next setup
![](/images/pic/zabbix/zabbix-config003.png)
上图中配置好之后，继续点击 Next setup
![](/images/pic/zabbix/zabbix-config004.png)
上图中，name尽量取有意义的名字，继续点击 Next setup
![](/images/pic/zabbix/zabbix-config005.png)
到这一步可以看到全部配置，确认无误后点击 Next setup
![](/images/pic/zabbix/zabbix-config006.png)
登录zabbix
![](/images/pic/zabbix/zabbix-config007.png)
登录之后点击 `管理-用户-点击Admin` ，可以设置超级管理基本属性，例如语言和主题，点击`配置-主机` ，可以看到如下图，接下来安装zabbix客户端
![](/images/pic/zabbix/zabbix-config008.png)
## 安装zabbix agent客户端
>这里的客户端作用是监控服务端本机

配置客户端，配置文件/etc/zabbix/zabbix_agentd.conf
```bash
# 主要配置如下，默认即可
Server=127.0.0.1
ServerActive=127.0.0.1
Hostname=Zabbix server
# 启动zabbix客户端
systemctl start zabbix-agent
# 开机启动
systemctl enable zabbix-agent
```
现在可以看到`可用性ZBX` 为绿色，表示的是zabbix-agent服务连接正常
![](/images/pic/zabbix/zabbix-config009.png)