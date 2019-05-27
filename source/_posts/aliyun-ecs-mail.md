layout: post
title: 阿里云ECS用465端口发邮件
author: Pyker
categories: Linux
img: /images/bar/aliyun.jpg
tags:
  - aliyun
top: false
cover: false
date: 2018-03-25 22:32:00
---
### 原由
大家可能都知道在阿里云购买的ECS云主机是不能直接通过25号端口发邮件的，因为阿里云底层对25号端口做了屏蔽。所以例如我们需要做监控、报告等邮件通知行为时，只能修改默认的25号端口。
下面我们就开始如何用465端口来发送邮件。

### 请求数字证书
* 创建目录，用来存放证书

	  [root@PLAY ~]# mkdir -p /root/.certs/
	  [root@PLAY ~]# echo -n | openssl s_client -connect smtp.126.com:465 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ~/.certs/126.crt
	  depth=2 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Global Root CA
	  verify return:1
	  depth=1 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = GeoTrust RSA CA 2018
	  verify return:1
	  depth=0 C = CN, L = Hangzhou, O = "NetEase (Hangzhou) Network Co., Ltd", OU = Mail Dept., CN = *.126.com
	  verify return:1
	  DONE
> `smtp.126.com:465` 为发件者的邮件服务器，我这用的是网易`126.com` 邮箱。生成`126.crt` 的证书。

* 添加一个证书到证书数据库中

	  [root@PLAY ~]# certutil -A -n "GeoTrust SSL CA" -t "C,," -d ~/.certs -i ~/.certs/126.crt
	  [root@PLAY ~]# certutil -A -n "GeoTrust Global CA" -t "C,," -d ~/.certs -i ~/.certs/126.crt
	 
* 列出目录下证书

	  [root@PLAY ~]# certutil -L -d /root/.certs
	  Certificate Nickname Trust Attributes
	  SSL,S/MIME,JAR/XPI
	  GeoTrust SSL CA C,,
### 获取126邮箱的授权码
前往邮箱的登陆网站登陆自己的邮箱，在邮箱设置里找到`POP3/SMTP/IMAP` 设置，并且完成客户端授权密码。如下图：
![邮箱授权码](/images/pic/shouquanma.png)

### 配置/etc/mail.rc文件
在/etc/mail.rc文件末尾追加如下参数
```
set bsdcompat
set from=test_wly@126.com
set smtp="smtps://smtp.126.com:465"
set smtp-auth-user=test_wly@126.com
set smtp-auth-password=73jdi9dw7j3gc8gf1xvak01fss
set smtp-auth=login
set ssl-verify=ignore
set nss-config-dir=/root/.certs
```

### 测试邮件发送是否正常
```
[root@PLAY ~]# echo "test mail" | mail -s "nagios report" test_wly@126.com
```
>如果配置正确，此时test_wly@126.com就能收到刚刚发送的测试邮件了。

看起来已经成功了，但是发送完邮件还有报错：证书不被信任，且命令行就此卡住，需要按键才能出现命令提示符
`Error in certificate: Peer's certificate issuer is not recognized.` 
可以按如下操作即可解决问题：
```
[root@PLAY ~]# cd /root/.certs/
[root@PLAY .certs]# certutil -A -n "GeoTrust SSL CA - G3" -t "Pu,Pu,Pu" -d ./ -i 126.crt 
Notice: Trust flag u is set automatically if the private key is present.
```