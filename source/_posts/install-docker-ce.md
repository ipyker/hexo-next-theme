layout: post
title: CentOS7 安装Docker-CE
author: Pyker
categories: docker
tags:
  - docker
  - kubernetes
top: false
cover: false
date: 2019-03-21 22:02:00
---
### CentOS7 安装Docker-CE

从2017年3月开始 docker 在原来的基础上分为两个分支版本: {% label primary@Docker-CE %} 和 {% label success@Docker-EE %}。`Docker-CE` 即社区免费版，`Docker-EE` 即企业版，强调安全，但需付费使用。本文介绍 Docker-ce的安装使用。

#### 移除旧的版本
```bash
	$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```
#### 安装一些必要的系统工具
```bash
	$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
#### 添加软件源信息					
```bash
	$ sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
#### 更新 yum 缓存
```bash
	$ sudo yum makecache fast
```
#### 查看可用版本的 Docker-CE
```bash
	$ yum list docker-ce --showduplicates | sort -r
```
<div class="note primary">注意：如果需要只显示table版本，可以关闭测试版本的list</p></div>

```bash
$ sudo yum-config-manager --enable docker-ce-edge
$ sudo yum-config-manager --enable docker-ce-test
```
#### 更新yum包索引
```bash
	$ yum makecache fast
```
#### 安装指定版本的docker-CE
```bash
	$ sudo yum install -y docker-ce-17.03.2.ce-1.el7.centos 
```
<div class="note warning"><p>报错：如果在安装指定版本的docker时显示需要安装指定版本的docker-ce-selinux依赖包，请安装：</p></div>

```bash
$ yum install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm 
```