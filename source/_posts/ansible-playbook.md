layout: post
title: Ansible playbook详解
author: Pyker
categories: CI/CD
img: /images/bar/playbook.jpg
tags:
  - ansible
top: false
cover: false
date: 2018-03-23 10:20:00
---
## Playbook简介
Playbooks与Ad-Hoc相比，是一种完全不同的运用Ansible的方式，而且是非常之强大的；也是系统ansible命令的集合，其利用[yaml语法](http://www.ansible.com.cn/docs/YAMLSyntax.html)编写，运行过程，ansbile-playbook命令根据`自上而下的顺序`依次执行任务。playbook 由一个或多个 ‘plays’ 组成.它的内容是一个以 ‘plays’为元素的列表，在 play 之中,一组机器被映射为定义好的角色.在 ansible 中,play 的内容,被称为 tasks,即任务.在基本层次的应用中,一个任务是一个对 ansible模块的调用。当第一个任务依次在所有主机上执行完毕后，开始执行第二个任务。如果某个主机执行时发生错误，则所有操作将会回滚。

## Playbook基础组件
* `hosts`：运行执行任务（task）的目标主机
* `remote_user`：在远程主机上执行任务的用户
* `tasks`：任务列表
* `handlers`：任务，与tasks不同的是只有在接受到通知时才会被触发
* `templates`：使用模板语言的文本文件，使用jinja2语法。
* `variables`：变量，变量替换{{ variable_name }}
* `tag`：标签，为某tasks指定标签，运行该标签可以即运行特定的tasks，定义为always的tag总会执行
* `when`: 条件判断，当条件成立则执行tasks，不成立不执行表达式 判断表达式如：`not` `or` `and` `!=` `=`
* `with_items`：循环迭代需要重复执行的任务列表，用`{item}}`引用列表值

例如一个简单的playbook文件：
```yaml
 ---
    - hosts: test                                  # 指定运行任务的主机组
      remote_user: root                            # 指定远程执行任务的用户
      vars:                                        # 指定变量
        - bsh: b.sh
        - httprpm: httpd
      task:                                        # 任务的开始
        - name: install httpd                      # 一个安装httpd的任务
          yum: name={{ httprpm }} state=present    # ansible的yum模块
          tags: install_httpd                      # 为该任务打一个install_httpd的标签
        - name: copy b.sh                          # 又一个复制b.sh脚本的任务
          copy: src=/root/{{ bsh }} dest=/root/ owner=ala group=ala mode=0644
          notify:                                  # 如果copy的文件内容发生改变就会触发
            - reload httpd                         # 指定通知的哪个handlers
          when: ansible_distribution == "CentOS"   # 通过变量判断系统为CentOS时才执行该copy任务
        - name: copy b.sh                          
          copy: src=/root/{{ bsh }} dest=/opt/ owner=ala group=ala mode=0644
          notify:                                  
            - reload httpd                         
          when: ansible_distribution == "Ubuntu"   # 通过变量判断系统为Ubuntu时才执行该copy任务
        - name: start httpd                        # 又一个启动httpd服务的任务
          service: name=httpd state=started enabled=yes
      handlers:                                    # 满足触发条件则执行的任务
        - name: reload httpd                       # 满足触发条件的任务名
          service: name=httpd state=reloaded       # 该任务为重新加载一下httpd服务
```
> 其中的`ansible_distribution`是ansible收集的facts变量。

## playbook定义变量
** ansible 常见定义变量有以下 6 种**
* /etc/ansible/hosts文件主机中定义
* /etc/ansible/hosts/文件主机组中定义
* playbook的yaml文件中通过vars定义
* 获取系统变量，也称facts变量
* 分文件定义主机和主机组的变量
* playbook 的role中定义


```bash
# /etc/ansible/hosts文件主机中定义 主机变量
192.168.200.136 http_port=808 maxRequestsPerChild=808
192.168.200.137 http_port=8080 maxRequestsPerChild=909
 
# /etc/ansible/hosts文件主机中定义 主机组变量
[websers]
192.168.200.136
192.168.200.137
 
[websers:vars]  
ntp_server=ntp.exampl.com
proxy=proxy.exampl.com                # ntp_server和proxy变量可为websers组中主机使用

# playbook的yaml文件中通过vars定义
 ---
    - hosts: test                                  
      remote_user: root                            
      vars:                           # 定义bsh和httprpm的两个变量
        - bsh: b.sh
        - httprpm: httpd

# 获取系统facts变量
ansible 192.168.200.136 -m setup      # 可以获取主机所有的facts变量，通过{{variable_name}}引用

# 分文件定义主机和主机组的变量
/etc/ansible/group_vars/websers       # 定义主机组名为websers的变量文件
/etc/ansible/host_vars/hostpc         # 定义主机名为hostpc的变量文件

$ cat /etc/ansible/host_vars/hostpc   # 变量文件内容格式如下
---
ntp_server: acme.example.org
database_server: storage.example.org

# role中定义   （目录结构下文讲述）
$ cat /etc/ansible/roles/nginx/vars/main.yml
---
nginx_port: 80
nginx_domain: www.abc.com
nginx_user: nginx
```
## playbook role目录结构
### Roles简介
Ansible为了层次化、结构化地组织Playbook，使用了角色（roles）。Roles能够根据层次型结构自动装载变量文件、task及handlers等。简单来讲，roles就是通过分别将变量、文件、任务、模块及处理器放置于单独的目录中，并可以便捷地include它们，roles一般用于基于主机构建服务的场景中，但也可以用于构建守护进程等场景中。

### 创建Roles
创建roles时一般需要以下步骤：首先创建以roles命名的目录。然后在roles目标下分别创建以这个角色名称命令的目录，如websevers等，然后在每个角色命令的目录中分别创建files、handlers、tasks、templates、meta、defaults和vars目录，用不到的目录可以创建为空目录。最后在Playbook文件中调用各角色进行使用。

### roles内各目录含义解释
* `files`：用来存放由copy模块或script模块调用的文件。
* `templates`：用来存放jinjia2模板，template模块会自动在此目录中寻找jinjia2模板文件。
* `tasks`：此目录应当包含一个main.yml文件，用于定义此角色的任务列表，此文件可以使用include包含其它的位于此目录的task文件。
* `handlers`：此目录应当包含一个main.yml文件，用于定义此角色中触发条件时执行的动作。
* `vars`：此目录应当包含一个main.yml文件，用于定义此角色用到的变量。
* `defaults`：此目录应当包含一个main.yml文件，用于为当前角色设定默认变量。
* `meta`：此目录应当包含一个main.yml文件，用于定义此角色的特殊设定及其依赖关系。

如下定义了一个nginx服务的playbook目录结构树。
```bash
[root@k8s-master1 roles]# tree /etc/ansible/roles/nginx
/etc/ansible/roles/nginx
├── defaults
│   └── main.yml
├── files
│   └── nginx.tar.gz
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
│   └── nginx.conf.j2
│   └── default.conf.j2
└── vars
    └── main.yml
```

## 使用roles安装nginx案例
### 实例环境
主机名 | IP | 系统 | 角色
--- | --- | --- | ---
ansible | 192.168.20.210 | centos7.5 | ansible控制器
web-nginx | 192.168.20.213 | centos7.5 | web服务器 

### hosts主机组清单
```bash
$ cat /etc/ansilbe/hosts          # ansible hosts主机组配置
  [web-node]
  192.168.20.213
```
### 生成ssh密钥
```bash
# 在ansible控制机上生成ssh密钥对
 
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:m4+23Eh+dJ8r/9zSjpUbHJECh1iFQU8Z0QMlygrRm98 root@k8s-node1
The key's randomart image is:
+---[RSA 2048]----+
|       .. +==OB. |
|        .o.o*..o.|
|       .  oo o o.|
|        .o.   . .|
|        S.. .  . |
|         o...E. o|
|        +. . . *.|
|       +.=. . ++=|
|       .*oo  o+*=|
+----[SHA256]-----+

# 复制ansible控制机上的公钥到nginx web服务器

$ ssh-copy-id -i ~/.ssh/id_rsa.pub 192.168.20.213

这里是root用户验证，输入完root密码后，就可以在ansible主机上无密码ssh登陆nginx web服务器了。
```
### 创建nginx的roles结构目录
```bash
$ mkdir -pv /etc/ansible/roles/nginx/{defaults,files,handlers,meta,tasks,templates,vars}
```
### 准备安装包和依赖包
```bash
cd /etc/ansible/roles/nginx/files
$ wget http://nginx.org/download/nginx-1.16.0.tar.gz
$ wget https://www.openssl.org/source/openssl-1.0.2r.tar.gz
$ wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.42.tar.gz
$ wget http://www.zlib.net/zlib-1.2.11.tar.gz
```
### 准备安装脚本
>该脚本已在手动安装测试过，建议各位在通过ansible playbook安装服务器前，先手动安装一遍确保无误。

```bash
$ cat /etc/ansible/roles/nginx/files/install_nginx.sh
#!/bin/sh
#info: source code install nginx 1.16.0
#author: pyker <pyker@qq.com>
export PATH=`echo $PATH`
PWD="/opt"
NGINX=nginx-1.16.0
OPENSSL=openssl-1.0.2r
PCRE=pcre-8.42
ZLIB=zlib-1.2.11
RUN_USER=nginx
id -u ${RUN_USER} >/dev/null 2>&1
[ $? -ne 0 ] && useradd -M -s /sbin/nologin ${RUN_USER}
mkdir -p /var/log/nginx
yum install -y epel-release 
yum install -y jemalloc jemalloc-devel
yum install -y gcc \
             gcc-c++ \
             gcc++ \
             perl \
             perl-devel \
             perl-ExtUtils-Embed \
             libxslt \
             libxslt-devel \
             libxml2 \
             libxml2-devel \
             gd \
             gd-devel \
             GeoIP \
             GeoIP-devel
cd $PWD
for tar in `ls *.tar.gz`; do
  tar zxf $tar
done

cd $PWD/$NGINX && ./configure --prefix=/usr/local/nginx \
            --sbin-path=/usr/sbin/nginx \
            --user=nginx \
            --group=nginx \
            --pid-path=/var/run/nginx.pid \
            --lock-path=/var/run/nginx.lock \
            --error-log-path=/var/log/nginx/error.log \
            --http-log-path=/var/log/nginx/access.log \
            --with-select_module \
            --with-poll_module \
            --with-threads \
            --with-file-aio \
            --with-http_ssl_module \
            --with-http_v2_module \
            --with-http_realip_module \
            --with-http_addition_module \
            --with-http_xslt_module=dynamic \
            --with-http_image_filter_module=dynamic \
            --with-http_geoip_module=dynamic \
            --with-http_sub_module \
            --with-http_dav_module \
            --with-http_flv_module \
            --with-http_mp4_module \
            --with-http_gunzip_module \
            --with-http_gzip_static_module \
            --with-http_auth_request_module \
            --with-http_random_index_module \
            --with-http_secure_link_module \
            --with-http_degradation_module \
            --with-http_slice_module \
            --with-http_stub_status_module \
            --with-mail=dynamic \
            --with-mail_ssl_module \
            --with-stream \
            --with-stream_ssl_module \
            --with-stream_realip_module \
            --with-stream_geoip_module=dynamic \
            --with-stream_ssl_preread_module \
            --with-compat \
            --with-ld-opt="-ljemalloc" \
            --with-pcre=../${PCRE} \
            --with-pcre-jit \
            --with-zlib=../${ZLIB} \
            --with-openssl=../${OPENSSL}\
            --with-openssl-opt=no-nextprotoneg \
            --with-debug && make -j 4 && make install
```
 ### 配置变量
 在安装nginx的时候或者其他服务的时候常常用到变量，下面配置变量是通过roles方式配置。
 ```bash
$ cat /etc/ansible/roles/nginx/vas/main.yml
cat nginx/vars/main.yml 
nginx_user: nginx
nginx_port: 80
nginx_dir: /usr/local/nginx
 ```
 ### 准备nginx.conf模版
 ```bash
$ cat /etc/ansible/roles/nginx/templates/nginx.conf.j2
user {{ nginx_user }};
worker_processes {{ ansible_processor_vcpus }};

error_log /var/log/nginx/error_nginx.log crit;
pid /var/run/nginx.pid;
worker_rlimit_nofile 51200;

events {
  use epoll;
  worker_connections 51200;
  multi_accept on;
}

http {
  map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
  }
  include mime.types;
  default_type application/octet-stream;
  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 1024m;
  client_body_buffer_size 10m;
  sendfile on;
  tcp_nopush on;
  keepalive_timeout 120;
  server_tokens off;
  tcp_nodelay on;

  #Gzip Compression
  gzip on;
  gzip_buffers 16 8k;
  gzip_comp_level 6;
  gzip_http_version 1.1;
  gzip_min_length 256;
  gzip_proxied any;
  gzip_vary on;
  gzip_types
    text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
    text/javascript application/javascript application/x-javascript
    text/x-json application/json application/x-web-app-manifest+json
    text/css text/plain text/x-component
    font/opentype application/x-font-ttf application/vnd.ms-fontobject
    image/x-icon;
  gzip_disable "MSIE [1-6]\.(?!.*SV1)";

  #If you have a lot of static files to serve through Nginx then caching of the files' metadata (not the actual files' contents) can save some latency.
  open_file_cache max=1000 inactive=20s;
  open_file_cache_valid 30s;
  open_file_cache_min_uses 2;
  open_file_cache_errors on;

########################## vhost #############################
  include upstream/*.conf;
  include vhost/*.conf;
}
 ```
 >nginx.conf.j2文件中`nginx_user`和`ansible_processor_vcpus`分别为用户自定义变量和`-m setup`获取的facts。

 ### 定义默认server模版
 ```bash
 ```bash
$ cat /etc/ansible/roles/nginx/templates/default.conf.j2
server {
        listen {{ nginx_port }};
        server_name {{ ansible_all_ipv4_addresses }};
        index index.html index.htm index.jsp index.do;
        access_log  off;

        location / {
            root /data/wwwroot;
        }

         location /nginx_status {
            stub_status on;
            access_log   off;
  }
}
 ```
 >default.conf.j2文件中`nginx_port`和`ansible_all_ipv4_addresses`分别为用户自定义变量和`-m setup`获取的facts。

### 定义一个测试index.html文件
```bash
$ cat /etc/ansible/roles/nginx/templates/index.html
 this is test pages!
```
### 定义nginx启动脚本
```bash
$ cat /etc/ansible/roles/nginx/files/nginx.service
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/usr/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
### 定义tasks任务
```yaml
# 在tasks目录中定义了一个copy.yaml文件用户复制files目录下的文件到nginx服务器
# 定义完成后，需要在tasks/main.yaml文件中用-include包含该copy.yaml

$ cat /etc/ansible/roles/nginx/tasks/copy.yaml
- name: copy nginx-1.16.0.tar.gz to client
  copy: src=/etc/ansible/roles/nginx/files/nginx-1.16.0.tar.gz dest=/opt/nginx-1.16.0.tar.gz
- name: copy install_nginx.sh to client
  copy: src=/etc/ansible/roles/nginx/files/install_nginx.sh dest=/opt/install_nginx.sh
- name: mkdir nginx data directory
  file: path=/data/wwwroot state=directory recurse=yes
- name: copy index.html
  copy: src=/etc/ansible/roles/nginx/files/index.html dest=/data/wwwroot/index.html
- name: copy nginx systemctl file
  copy: src=/etc/ansible/roles/nginx/files/nginx.service dest=/usr/lib/systemd/system/nginx.service
- name: copy dependency package
  copy: src=/etc/ansible/roles/nginx/files/{{ item }} dest=/opt/{{ item }}
  with_items:
    - openssl-1.0.2r.tar.gz
    - pcre-8.42.tar.gz
    - zlib-1.2.11.tar.gz
```
```yaml
# 定义main.yaml文件

$ cat /etc/ansible/roles/nginx/tasks/main.yml 
---
- include: copy.yml       # 使用include将上面copy.yaml文件包含进来
- name: install nginx
  shell: /usr/bin/sh /opt/install_nginx.sh
- name: replace nginx.conf file
  template: src=/etc/ansible/roles/nginx/templates/nginx.conf.j2 dest={{ nginx_dir }}/conf/nginx.conf
- name: mkdir nginx vhost directory
  file: path={{nginx_dir}}/conf/vhost state=directory
- name: relpace default.conf file
  template: src=/etc/ansible/roles/nginx/templates/default.conf.j2 dest={{nginx_dir}}/conf/vhost/default.conf
  tags: ngxdef            # 定义一个tag，当default.conf发生改变时可以指定运行该任务
  notify:
    - reload nginx        # 该名称需要和handler中定义的一样
- name: start nginx
  command: /usr/sbin/nginx # 由于配置了nginx启动服务，也可以使用 shell: systemctl start nginx
```
### 定义触发通知handlers
```yaml
$ cat /etc/ansible/roles/nginx/handlers/main.yml 
---
- name: reload nginx
  shell: /usr/sbin/nginx -t; /usr/sbin/nginx -s reload  # 也可以使用 shell: systemctl reload nginx
```
### 定义role入口文件
```bash
$ cat /etc/ansible/roles/nginx.yml 
---
- hosts: web-node      # 定义要执行该playbook的主机组
  remote_user: root    # 定义远程执行命令的用户
  roles:               # 
    - nginx            # 主机组中的主机使用哪个playbook剧本
```
### 查看当前nginx roles目录树结构
```bash
$ tree /etc/ansible/roles/
/etc/ansible/roles/
├── nginx
│   ├── defaults
│   ├── files
│   │   ├── index.html
│   │   ├── nginx.service
│   │   ├── install_nginx.sh
│   │   ├── nginx-1.16.0.tar.gz
│   │   ├── openssl-1.0.2r.tar.gz
│   │   ├── pcre-8.42.tar.gz
│   │   └── zlib-1.2.11.tar.gz
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   ├── tasks
│   │   ├── copy.yml
│   │   └── main.yml
│   ├── templates
│   │   ├── default.conf.j2
│   │   └── nginx.conf.j2
│   └── vars
│       └── main.yml
└── nginx.yml
```
### playbook语法检测
通过上面的配置，我们已经完成了使用playbook方式安装nginx所需要的步骤，现在我们应该在执行该playbook前检查一下上述语法有没有错误。
```bash
$ ansible-playbook --syntax-check /etc/ansible/roles/nginx.yml 

playbook: /etc/ansible/roles/nginx.yml  # 如果语法没问题，将直接显示文件名称，如果有错误将告知你那里错了
```

### 测试安装
`ansible-playbook -C` 命令可以测试运行playbook剧本，而非真正在远端服务器上执行，这样可以方便查看playbook在执行过程中将会做哪些事情。
```bash
$ ansible-playbook -C nginx.yml 

PLAY [web-node] ************************************************************************************************************************************************

TASK [nginx : copy nginx-1.16.0.tar.gz to client] **************************************************************************************************************
changed: [192.168.20.213]

TASK [nginx : copy install_nginx.sh to client] *****************************************************************************************************************
changed: [192.168.20.213]

TASK [nginx : mkdir nginx data directory] **********************************************************************************************************************
changed: [192.168.20.213]

TASK [nginx : copy index.html] *********************************************************************************************************************************
changed: [192.168.20.213]

TASK [nginx : copy nginx systemctl file] ***********************************************************************************************************************
changed: [192.168.20.213]

TASK [nginx : copy dependency package] *************************************************************************************************************************
changed: [192.168.20.213] => (item=openssl-1.0.2r.tar.gz)
changed: [192.168.20.213] => (item=pcre-8.42.tar.gz)
changed: [192.168.20.213] => (item=zlib-1.2.11.tar.gz)

TASK [nginx : install nginx] ***********************************************************************************************************************************
skipping: [192.168.20.213]

TASK [nginx : replace nginx.conf file] *************************************************************************************************************************
changed: [192.168.20.213]

TASK [nginx : mkdir nginx vhost directory] *********************************************************************************************************************
changed: [192.168.20.213]

TASK [nginx : relpace default.conf file] ***********************************************************************************************************************
changed: [192.168.20.213]

TASK [nginx : start nginx] *************************************************************************************************************************************
skipping: [192.168.20.213]

RUNNING HANDLER [nginx : reload nginx] *************************************************************************************************************************
skipping: [192.168.20.213]

PLAY RECAP *****************************************************************************************************************************************************
192.168.20.213             : ok=8    changed=8    unreachable=0    failed=0
```
### 运行nginx playbook
```bash
$ ansible-playbook nginx.yml
```
>执行`ansible-playbook nginx.yml`命令后，ansible将会按照刚刚测试安装的步骤在远端进行安装nginx服务并且启动。

### 验证安装结果
等待一会nginx playbook将会在远端服务器上安装完毕，现在我们来验证一下结果：
```bash
# 在web服务器上查看nginx端口
$ netstat -ptln | grep 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      14461/nginx: master

# 查看nginx进程
$ ps -ef | grep nginx
root      14461      1  0 00:39 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx     14474  14461  0 00:39 ?        00:00:00 nginx: worker process
nginx     14475  14461  0 00:39 ?        00:00:00 nginx: worker process
nginx     14476  14461  0 00:39 ?        00:00:00 nginx: worker process
nginx     14477  14461  0 00:39 ?        00:00:00 nginx: worker process
root      39501   1664  0 11:38 pts/0    00:00:00 grep --color=auto nginx

# 让我们在内网任意一台电脑上来访问一下该nginx页面
$ curl http://192.168.20.213
this is a test pages!       # 结果是我们前面定义的index.html内容

# 访问一下nginx_status的uri
$ curl http://192.168.20.213/nginx_status     #前面default.conf.j2文件中定义的nginx location
Active connections: 1 
server accepts handled requests
 8 8 10 
Reading: 0 Writing: 1 Waiting: 0
```
至此，一个使用ansible-playbook安装nginx服务已完成。