layout: post
title: Ansible 常用模块指令
author: Pyker
categories: CI/CD
img: /images/bar/ansible.jpg
tags:
  - ansible
top: false
cover: false
date: 2018-03-22 14:47:00
---
>**本文主要讲解ansible的常用命令和简单安装步骤，具体配置文件详解以及playbook暂未涉及！后续更新。。。。**

### 安装Ansible
```bash
$ yum install epel-release ansible -y
```
### 配置文件配置
* 常用ansible.cfg配置文件
```bash
$ cat /etc/ansible/ansible.cfg 
  [defaults]
  inventory      = /etc/ansible/hosts           #主机清单
  library        = /usr/share/my_modules/       # 使用的模块
  forks          = 5                            #处理并发的进程数，建议设置控制机的数量
  sudo_user      = root                         #默认执行远程命令的用户
  remote_port    = 22                           #SSH连接被控制机的端口
  gathering = smart                             #'smart'收集facts的信息，如已收集，则采用缓存。可选参数'implicit'、'explicit'，分别表示每次都收集和默认不收集facts信息
  fact_caching_timeout = 86400                  #'gathering为smart'可用，设置收集超时时间
  fact_caching = jsonfile                       #'gathering为smart'可用，设置以什么存储facts信息。还可在本地安装redis、memcache来作为facts的存储。
  fact_caching_connection = /etc/ansible/ansible_facts_cache  #'gathering为smart'可用，设置缓存路径。
  roles_path    = /etc/ansible/roles            #设置ansible其他roles的路径
  host_key_checking = false                     #忽略检查主机密钥
  log_path = /var/log/ansible.log               #ansible日志路径
  module_name = command                         #ansible默认的模块
  deprecation_warnings = False                  #禁用ansible“不建议使用”的警告

  [ssh_connection]
  ssh_args = -C -o ControlMaster=auto -o ControlPersist=1d        #开启ssh长连接，保存时间为1天
  control_path_dir = /etc/ansible/ssh-socket                      #ssh长连接存放的路径
  control_path = %(directory)s/%%h-%%p-%%r                        #ssh长连接的格式名称
  pipelining = True                                               #减少执行远程模块SSH操作次数，开启这个设置,将显著提高性能，但被控制机需要将/etc/sudoers下"Defaults requiretty"注释掉

  [accelerate]
  accelerate_port = 5099                        #使用python程序在被控制机上运行一个守护进程，ansible通过这个守护进程监听的端口进行通信。所有机器都要安装python-keyczar包
  accelerate_timeout = 30                       #设置用来控制从客户机获取数据的超时时间，如果在这段时间内没有数据传输,套接字连接会被关闭。
  accelerate_connect_timeout = 5.0              #设置空着套接字调用的超时时间.这个应该设置相对比较短.这个和`accelerate_port`连接在回滚到ssh或者paramiko连接方式之前会尝试三次开始远程加速daemon守护进程.默认设置为1.0秒:
```
* hosts 文件
 （机器清单，进行分组管理）
 ```bash
  [tomcat]
  192.168.16.192
  192.168.16.193
  [nginx]
  app1 ansible_ssh_host=192.168.16.190 ansible_ssh_pass="2039"
  app2 ansible_ssh_host=192.168.16.191 sudo_user=”ald” ansible_ssh_pass="2039"
```
>*sudo_user和ansible_ssh_pass为帐号密码方式验证，建议用密钥。*
### Ansible指令参数
* __ansible 常用命令参数__
-m指定模块
-a 指定模块的命令。默认是command模块，可以省略
-B 指定ansible后台运行超时时间
-C 测试运行效果，而不是正在运行
-f 指定使用的并行进程的数量
-i 指定inventory/hosts文件，默认/etc/ansiable/hosts文件
--limit=xxx.xxx.xxx.xxx 限制对某个ip或者网段或者组执行
--list-hosts 显示将要执行命令的主机

* __ansible-doc 常用命令参数__
-M --module-path=/xxx/xxx  查询模块 默认是/usr/share/ansible/
-l  --list          显示已存在的所有模块
-s  command     显示playbook制定模块的用法，类似 man 命令

* __ansible-galaxy__ 
>下载第三方模块指令，类似yum、pip、easy_install这样的命令

   ansible-galaxy install <module_name>
   
* __ansible-playbook__ 常用命令参数
--syntax-check  [yaml文件]         语法检测
-t TAGS    只允许指定的tags标签任务，多个以 , 分开
--skip-tags=SKIP_TAGS   跳过指定的标签
--start-at-task=START_AT   从哪个任务后执行

### 常用指令模块
1、 copy——拷贝模块   (用于将本地或远程机器上的文件拷贝到远程主机上)
```bash
$ ansible all –m copy -a “src=/xxx/xxx dest=/yyy/yyy owner=root group=root mode=644 force=yes/no backup=yes/no”
解释：将src 文件/目录复制到远程dest上，所有者/组为root 权限为644，force为是否强制替换，backup为替换前是否需要备份远程远文件
```

2、 raw——命令模块 （和command、shell类似）
```bash
$ ansible all –m raw -a "ifconfig"
解释：在所有主机上执行ifocnfig命令。
```

3、 yum——安装模块 （安装程序）
```bash
$ ansible all –m yum –a “name=httpd state=present”
解释：安装httpd程序，state可以是present、latest、installed表示安装程序，absent、removed表示卸载程序
```

4、 file——文件模块 （文件属性修改）
```bash
$ ansible all –m file –a “src=/xxx/xxx/1 dest=/yyy/1 state=link owner=alad group=alad mode=777”
$ ansible develop -m file -a "path=/xxx/dir recurse=yes owner=root group=alad mode=644"
解释：1、将src的文件软连接到dest目录下，并修改所有者/组和权限
      2、将path路径的目录递归形式设置所有者和权限
*、state还可以是directory：如果目录不存在，创建目录
    file：即使文件不存在，也不会被创建
	hard：创建硬链接
    touch：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间
    absent：删除目录、文件或者取消链接文件
```

5、 cron——计划任务模块 （计划任务crontab）
```bash
$ ansible develop -m cron -a "name='show time' minute=*/1 hour=* day=* month=* weekday=* job='/bin/date'"
$ ansible develop -m cron -a "name='show time' state=absent"
解释：1、创建一个每分钟显示时间的计划任务
      2、删除名为show time这个计划任务
```

6、 group——组模块（用户组）
```bash
$ ansible all-m group -a "name=develop"
解释：在所有主机上创建一个develop的组 ，state=absent表示删除该组
```

7、 user——用户模块（用户）
```bash
$ ansible develop -m user -a "name=harlan groups=root password=-1vFO89dP6qyK"
$ ansible develop -m user -a "name=harlan state=absent remove=yes"
解释：1、在所有主机上创建harlan用户，并将其添加到root组，密码是经过hash加密后的，明文密码会被哈希，所以先填入hash后的密码即可
可用此命令hash密码  openssl passwd -salt -1 "123456"
      2、删除harlan用户。Remove表示是否删除用户的同时删除家目录
```

8、 service——服务模块（服务状态）
```bash
$ ansible develop -m service -a "name=nginx state=running"
$ ansible develop -m service -a "name=nginx state=restarted enabled=yes"
解释：1、无论服务处在什么状态，最后都是将服务状态设置为启动，当服务正在运行的时候，显示为changed为false，state显示为状态，表示为正在运行；当服务停止的时候，显示为changed为true，表示这个时候将服务进行了启动，状态为启动
      2、表示重启nginx并且将nginx设置为开机自启动，state还有staeted、reloaded、stoped值
```

9、 script——脚本模块（运行脚本）
```bash
$ ansible develop -m script -a "/root/a.sh"
解释：在develop主机上运行当前服务器上的a.sh脚本
```

10、get_url——下载url上指定文件（类似wget）
```bash
$ ansible develop -m get_url -a "url=http://file.alavening.com/alading_file/head_img/1526900421976.jpg dest=/home/ owner=alad group=alad mode=644"
解释：将url上的图片下载到dest目的目录上,并且设置相应的所有者/组和权限。
```
 
11、synchronize——同步目录（默认推送，mode=pull为拉取）
```bash
$ ansible develop -m synchronize -a "src=/home/test/ dest=/home/test compress=yes delete=yes"
$ ansible develop -m synchronize -a "src=/home/test/ dest=/home/test compress=yes mode=pull"
解释：1、将src下的文件同步到dest上，delete=yes表示以src 目录为准镜像同步。
      2、拉取远程src上的目录文件到本地dest上
```
 
12、template——文档内变量的替换的模块
```bash
$ ansible develop –m template –a ‘src=/mytemplates/foo.j2 dest=/etc/file.conf mode="u=rw,g=r,o=r"’
解释：将src上foo.j2的变量模版复制到dest上。Template适合用playbook编写 ，通过变量然后拷贝到远程主机。
```
>可以参考：<https://www.cnblogs.com/jsonhc/p/7895399.html>

13、fetch——从远程主机下载文件（不能拉取目录）
```bash
$ ansible develop -m fetch -a "src=/home/test/xxx dest=/home/ flat=yes"
解释：将远程xxx文件拉取到本地home目录下，目录结构会是dest路径+远程主机名+src，假如远程主机名为develop，拉取的xxx文件在本地的/home/develop/home/test目录。如果需要指定拉取到某目录下 加个flat=yes的参数即可。
```
 
14、unarchive——解压文件
```bash
$ ansible develop -m unarchive -a "src=/root/apache-tomcat-7.0.85.tar.gz dest=/home/test owner=alad group=alad mode=755"
$ ansible develop -m unarchive -a "src=/home/alad/ansible/elk/logstash-6.2.4.tar.gz dest=/home/test remote_src=yes"
$ ansible develop -m unarchive -a "src=http://mirrors.linuxeye.com/oneinstack-full.tar.gz dest=/home/test remote_src=yes"
解释：将本地的tomcat压缩包解压到远程主机dest目录下，并修改其权限和所有者/组，remote_src=yes 表示解压远程主机已有的压缩包，src为url表示下载此包到远程主机dest目录进行解压缩后，并删除压缩包源文件
```
 
15、command和shell——linux命令模块
```
shell和command的区别：shell模块可以特殊字符，而command是不支持
简单说：command运行的命令中无法使用变量，管道。如果需要使用管道、变量，请使用raw模块,或者shell模块。
```
 
16、setup——获取主机信息
```bash
$ ansible develop -m setup
$ ansible develop -m setup -a 'filter=ansible_*_mb'
解释：1、显示系统所有信息
      2、通常配合filter进行过滤来获取主机信息，（例子是显示内存信息）
```

17、assemble——配置文件组装发送到远程主机
```bash
$ ansible test -m assemble -a "src=/root/test dest=/root/ansible/fileone mode=777 remote_src=False delimiter='========'"
解释：将src test目录下所有文件（不含test子目录内容）的内容发送到dest fileone文件中，remote_src默认为Ture表示src为远程主机上的路径，False为ansible控制端的路径，delimiter为文件之间内容分隔符。
```

### Playbook语法和结构
Playbook需要7个文件夹，如ansible安装nginx，则需要在/etc/ansible/roles目录下建立以下文件夹。
mkdir -pv nginx/{default,tasks,vars,meta,handlers,templates,files}
对于Ansible，几乎每个YAML文件都以一个列表开始。列表中的每个项目都是键/值对列表，通常称为“散列”或“字典”。所以，我们需要知道如何在YAML中编写列表和字典。
YAML还有一个小小的怪癖。所有YAML文件（无论它们是否与Ansible相关联）都可以选择开始---和结束---.。这是YAML格式的一部分，并指示文档的开始和结束。
列表中的所有成员都是以相同的缩进级别开头的行（短划线和空格）："- "

例如：
```yaml
	---
	- hosts: test
	  remote_user: root
	  vars:
	    - bsh: b.sh
		- httprpm: httpd
	  task:
	    - name: install httpd
		  yum: name={{ httprpm }} state=present”
		  tags: install_httpd
		- name: copy b.sh
		  copy: src=/root/{{ bsh }} dest=/root/ owner=ala group=ala mode=0644
		  notify:
		    - reload httpd
		- name: start httpd
		  service: name=httpd state=started enabled=yes
	  handlers:
	    - name: reload httpd
		  service: name=httpd state=reloaded
```
第一次的话都会运行，后边如果copy的文件内容发生改变就会触发 notify ，然后会直接执行 handlers 的内容（ 这里notify后边的事件就都不会执行了 ）。

### template模块jinja2语法

* __template:使用了Jinjia2格式作为文件模版，进行文档内变量的替换的模块。相当于copy，将jinja2的文件模板理解并执行，转化为各个主机间的对应值。__
如：template: src=httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf

>http.conf.j2必须是完整的文件内容，因为这是覆盖操作，而非只选择性远程主机替换变量，dest要指定文件名，如果是目录就相当于copy了http.conf.j2到远程目录下，不是我们要的结果。

* __when语句：在tasks中使用，Jinja2的语法格式__

```yaml
- name: start nginx service
 shell: systemctl start nginx.service
 when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "7"
```
当系统为 centos 7的时候执行sysctemctl命令，否则不执行

* __循环：迭代，需要重复执行的任务__

>变量名为item，而with_item为要迭代的元素。如果某个任务出错，后面不执行
 
```yaml
- name: install packages
 yum: name={{ item }} state=latest
 with_items:
    - httpd
    - php
```

  >这是基于字符串列表给出元素示例 
  
```yaml
- name: create users
    user: name={{ item.name }} group={{ item.group }} state=present
    with_items:
    - {name: 'userx1', group: 'groupx1'}
    - {name: 'userx2', group: 'groupx2'}
```
这是基于字典列表给元素示例：item.name  . 后边的表示键。