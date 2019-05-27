layout: post
title: Tomcat8配置APR模式
author: Pyker
categories: web
img: /images/bar/tomcat.jpg
tags:
  - tomcat
top: false
cover: false
date: 2018-12-14 18:47:00
---
### tomcat三种模式
Tomcat Connector运行有三种模式：
* __bio__
默认的模式,同步阻塞，性能非常低下,没有经过任何优化处理和支持.

* __nio__ 
同步非阻塞，利用java的异步io护理技术,noblocking IO技术,想运行在该模式下，直接修改server.xml里的Connector节点,修改protocol为```protocol="org.apache.coyote.http11.Http11NioProtocol" ``` 启动后,就可以生效。

* __apr__
安装起来最困难,但是从操作系统级别来解决异步的IO问题,大幅度的提高性能. Tomcat apr也是在Tomcat上运行高并发应用的首选模式。必须要安装apr、apr-util和native，nio修改模式，修改protocol为 `org.apache.coyote.http11.Http11AprProtocol` ，直接启动就支持apr。

### 安装配置APR模式

**本文所有步骤的前提是已经可以正常运行tomcat程序，JDK已安装的环境。**

Tomcat配置apr模式依赖以下包,（ `版本根据自己需求选择` ）
```
apr-1.6.2
apr-util-1.6.0
openssl-1.0.2l
```
#### 下载依赖包
```bash
$ wget http://mirror.bit.edu.cn/apache//apr/apr-1.6.2.tar.gz
$ wget http://mirror.bit.edu.cn/apache//apr/apr-util-1.6.0.tar.gz
$ wget https://www.openssl.org/source/openssl-1.0.2l.tar.gz
```
#### 安装各个依赖包
```bash
#安装apr
$ tar zxvf apr-1.6.2.tar.gz
$ cd apr-1.6.2
$ ./configure --prefix=/usr/local/apr && make && make install
#安装apr-util
$ tar zxvf apr-util-1.6.0.tar.gz
$ cd apr-util-1.6.0
$ ./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr && make && make install
#安装openss-1.0.2l
$ tar zxvf openssl-1.0.2l.tar.gz
$ cd openssl-1.0.2l
$ ./config --prefix=/usr/local/openssl shared zlib && make && make install
```

*1、安装apr-util前请确认是否安装了 `expat-devel` 包，如没安装请安装，不然会报错。`yum install expat-devel` *

*2、检查openssl是否安装成功 `/usr/local/openssl/bin/openssl version -a`  显示1.0.2l版本为成功*

#### 安装tomcat-native
```bash
$ tar zxvf /usr/local/tomcat8/bin/tomcat-native.tar.gz
$ cd /usr/local/tomcat8/bin/tomcat-native-1.2.12-src/native
$ ./configure --with-apr=/usr/local/apr --with-java-home=/usr/local/java8/ --with-ssl=/usr/local/openssl/ && make && make install
```
#### 配置tomcat支持apr配置apr库文件
```bash
#方式1：配置坏境变量：
$ echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/apr/lib" >> /etc/profile
$ echo "export LD_RUN_PATH=$LD_RUN_PATH:/usr/local/apr/lib" >> /etc/profile && source /etc/profile
#
#方式2：catalina.sh脚本文件：在注释行# Register custom URL handlers下添加一行 
$ JAVA_OPTS="$JAVA_OPTS -Djava.library.path=/usr/local/apr/lib"
```
#### 修改tomcat server.xml文件

	 <Connector port="8080" protocol="org.apache.coyote.http11.Http11AprProtocol"
	 connectionTimeout="20000"
	 redirectPort="8443" />

#### 启动Tomcat
```bash
$ cd /usr/local/tomcat8/bin
$ ./startup.sh
```
#### 查看Tomcat模式运行
```bash
$ cat /usr/local/tomcat8/logs/catalina.out
```
>如果有显示`[http-apr-8080]` 说明配置APR模式成功。

### java.net.ConnectException异常处理
有时候在安装完tomcat后，停止tomcat会的会有如下异常，该异常可能和JDK有关系。
![](/images/pic/tomcat1.png)

解决办法：进入JDK目录，编辑java.security文件,(`注释掉原来的securerandom.source行，新增此行，保存即可` )
```bash
$ vi /usr/local/java8/jre/lib/security/java.security
securerandom.source=file:/dev/./urandom
```