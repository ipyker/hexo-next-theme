layout: post
title: Gitlab-CE社区版安装
author: Pyker
categories: CI/CD
img: /images/bar/gitlab.jpg
tags:
  - git
top: false
cover: false
date: 2018-11-29 17:30:00
---
常见的git环境有git web版和gitolite版本，gitolite权限控制强大，但功能不完善，而git web(gitlab)虽然权限没有像gitolite分的那么细，但是功能异常强大，不仅支持SSH KEY、web密码方式还有CI/CD，PIPELINE功能。所以下文将介绍如何在公司内网搭建私有的gitlab环境。

>GitLab官网强烈建议安装Omnibus软件包，因为它安装更快，更易于升级，并且包含增强其他方法所没有的可靠性的功能。我们还强烈建议`至少4GB`的可用内存` 来运行GitLab。

## GitLab的安装

### Omnibus方式安装

##### 安装并配置必要的依赖项
在CentOS 7（和RedHat / Oracle / Scientific Linux 7）上，执行以下命令。
```bash
sudo yum install -y curl policycoreutils-python openssh-server
```
##### 更新GitLab国内yum源
```bash
$ cat > /etc/yum.repos.d/gitlab-ce.repo << EOF
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1 
EOF
```
##### 安装GitLab-CE包
```bash
$ yum makecache
$ yum install gitlab-ce -y
```
##### 配置GitLab-CE域名和邮件通知
配置文件为`/etc/gitlab/gitlab.rb` 
```bash
#修改external_url 为你gitlab的访问域名
$ vim /etc/gitlab/gitlab.rb
external_url 'http://git.ipyker.com'

#添加gitlab邮件通知配置（大约在/etc/gitlab/gitlab.rb文件的517行）
gitlab_rails['smtp_enable'] = true 
gitlab_rails['smtp_address'] = "smtp.ipyker.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "gitlab@ipyker.com"
gitlab_rails['smtp_password'] = "mypasswd"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true 
gitlab_rails['smtp_tls'] = true 
gitlab_rails['gitlab_email_from'] = 'gitlab@ipyker.com'
gitlab_rails['smtp_domain'] = "ipyker.com"
```

> **GitLab安装完后，工作目录默认为：/var/opt/gitlab、/opt/gitlab，配置文件目录为：/etc/gitlab**

#### GitLab管理
##### 重载配置信息
```
$ gitlab-ctl reconfigure
```
##### 启动
```
$ gitlab-ctl start
```
##### 停止
```
$ gitlab-ctl stop
```
##### 重启 
```
$ gitlab-ctl restart
```

#### 登陆
第一次用域名登录gitlab，需要为root用户修改密码，root用户也是gitlab的超级管理员，gitlab也支持修改中文界面，如下图所示
![](/images/pic/welcome-gitlab.png)

![](/images/pic/language.png)

**更多关于gitlab配置信息参数，请参考官网[GitLab配置文件](https://docs.gitlab.com/omnibus/settings/README.html)**

---
### Docker方式安装
#### GitLab docker 镜像
gitlab-ce镜像存放在[docker官方仓库](https://hub.docker.com/r/gitlab/gitlab-ce/)中。

GitLab Docker镜像是GitLab的单个镜像，在单个容器上运行所有必要的服务。
在以下示例中，我们使用GitLab CE的镜像。如果要使用最新的RC映像，请使用`gitlab/gitlab-ce:rc` 用于GitLab CE
GitLab Docker镜像可以多种方式运行：
* [在Docker Engine中运行映像](https://docs.gitlab.com/omnibus/docker/#run-the-image)
* [将GitLab安装到群集中](https://docs.gitlab.com/omnibus/docker/#install-gitlab-into-a-cluster)
* [使用docker-compose安装GitLab](https://docs.gitlab.com/omnibus/docker/#install-gitlab-using-docker-compose)

**本文档以第一种方式在docker中运行，如你需要高可用可以用第二种，甚至是通过kubenetes deplyment部署pod方式安装。**

#### 先决条件
需要先安装docker，请参考[docker-ce安装](https://www.ipyker.com/2019/03/21/centos7-install-docker-ce/)

#### 运行gitlab-ce容器
```bash
$ docker run -d \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```
>`docker run` 命令将直接拉取镜像和运行容器，相当于运行docker pull和docker start两条命令。gitlab-ce容器将制定域名，开放的端口，以及容器挂载本地的文件系统，所有GitLab数据都将存储为子目录 /srv/gitlab/。restart系统重启后，容器将自动运行。

宿主目录 | docker目录 | 用途
--- | --- | ---
/srv/gitlab/data | /var/opt/gitlab | 用于存储应用数据
/srv/gitlab/logs | /var/log/gitlab | 用于存储日志
/srv/gitlab/config | /etc/gitlab | 用于存储GitLab配置文件

#### 配置girlab-ce
此容器使用官方的Omnibus GitLab软件包，因此所有配置都在docker唯一的配置文件中完成/etc/gitlab/gitlab.rb，也可以通过修改挂载卷文件来完成。

要访问GitLab的配置文件，您可以在正在运行的容器的上下文中启动shell会话。这将允许您浏览所有目录并使用您喜欢的文本编辑器：
```bash
$ docker exec -it gitlab /bin/bash
```
打开后，请`/etc/gitlab/gitlab.rb` 确保将指针设置`external_url` 为有效的URL。

此外您可以通过将环境变量添加`GITLAB_OMNIBUS_CONFIG` 到docker run命令来预配置GitLab Docker映像。此变量可以包含任何gitlab.rb设置，并在加载容器gitlab.rb文件之前进行评估。这样，您可以轻松配置GitLab的外部URL，从[Omnibus GitLab](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/files/gitlab-config-template/gitlab.rb.template)模板进行任何数据库配置或任何其他选项 。
注意：包含的设置`GITLAB_OMNIBUS_CONFIG` 不会写入gitlab.rb配置文件，而是在加载时进行评估。
这是一个设置外部URL并在启动容器时启用LFS的示例：
```bash
$ docker run -d \
  --hostname gitlab.example.com \
  --env GITLAB_OMNIBUS_CONFIG="external_url 'http://my.domain.com/'; gitlab_rails['lfs_enabled'] = true;" \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```


**要从GitLab接收电子邮件，您必须配置 SMTP设置，因为GitLab Docker映像没有安装SMTP服务器。默认也为HTTP，还可以启动https访问。**

#### 重启gitlab-ce
完成所需的所有更改后，需要重新启动容器才能重新配置GitLab：
```bash
$ docker restart gitlab
```

#### 手动配置HTTPS
默认情况下，omnibus-gitlab不使用HTTPS。如果要为gitlab.example.com启用HTTPS，请将以下语句添加到`/etc/gitlab/gitlab.rb` ：
```
external_url "https://gitlab.example.com"
```
因为在我们的例子中，主机名是“gitlab.example.com”，所以创建`/etc/gitlab/ssl` 目录并在那里复制密钥和证书。
```bash
$ sudo mkdir -p /etc/gitlab/ssl
$ sudo chmod 700 /etc/gitlab/ssl
$ sudo cp gitlab.example.com.key gitlab.example.com.crt /etc/gitlab/ssl/
```

重新加载配置，当重新配置完成时，您的GitLab实例应该可以访问https://gitlab.example.com。
```bash
$ gitlab-ctl reconfigure
```
>如果certificate.key文件有设置密码，Nginx在`sudo gitlab-ctl reconfigure` 执行时不会要求输入密码。在这种情况下，Omnibus GitLab将无声地失败，没有错误消息。要从密钥中删除密码，请使用以下命令： `openssl rsa -in certificate_before.key -out certificate_after.key` 。