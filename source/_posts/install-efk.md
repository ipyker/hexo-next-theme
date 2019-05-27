layout: post
title: Elastic stack之EFK安装
author: Pyker
categories: elastic
img: /images/bar/kibana.jpg
tags:
  - elasticsearch
  - filebeat
  - kibana
top: false
cover: false
date: 2019-03-15 14:15:00
---
## 试验环境
本次试验的`elastic stack`软件都为`7.0.1`版本，现在官网最新的为7.1版本，用户可在官方下载[下载地址](https://www.elastic.co/cn/downloads/)。

主机名 | ip地址 | 服务
--- | --- | ---
efk-node1 | 192.168.20.211 | elasticsearch、kibana、jdk8
efk-node2 | 192.168.20.212 | elasticsearch、cerebro、jdk8
real-server | 192.168.20.250 | filebeat

## 环境准备
### 安装JDK
elasticsearch需要jdk环境的支持，7.0.1版本默认已经自带JDK了，但是为了日后扩展问题，我们还是手动配置一边JDK环境。以下操作在`efk-node1`和`efk-node2`主机上进行：
```bash
# 下面下载链接因授权问题，需用户前往JDK官网下载
$ wget https://download.oracle.com/otn/java/jdk/8u171-b12/478a62b7d4e34b78b671c754eaaf38ab/jdk-8u171-linux-x64.tar.gz

$ tar zxvf jdk-8u171-linux-x64.tar.gz -C /usr/local/

$ cat >> /etc/profile << EOF
export JAVA_HOME=/usr/local/jdk1.8.0_171
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
EOF

$ source /etc/profile

# 验证JDK, JDK配置完成
$ java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```
### 设置服务器的最大文件数
```bash
$ cat >> /etc/security/limits.conf << EOF
 * soft nofile 655350
 * hard nofile 655350
 * soft nproc 655350
 * hard nproc 655350
EOF
```
### 设置服务器打开的最大进程数
```bash
$ cat > /etc/security/limits.d/20-nproc.conf << EOF
*          soft    nproc     4096
root       soft    nproc     unlimited
EOF
```
### 设置nmap数量对虚拟内存的支持
```bash
$ cat >> /etc/sysctl.conf << EOF
vm.max_map_count=262144
EOF

$  sysctl -p
```
### 本地host解析
```bash
cat >> /etc/hosts << EOF
efk-node1 192.168.20.211
efk-node2 192.168.20.212
real-server 192.168.20.250
EOF
```
## 安装elasticsearch
以下操作在`efk-node1`和`efk-node2`主机上进行：
### 下载和解压
```bash
# 创建elastic工作目录
$ mkdir /elastic && cd /elastic

# 下载elasticsearch tar包
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.1-linux-x86_64.tar.gz

# 解压到当前/elastic目录
$ tar zxvf elasticsearch-7.0.1-linux-x86_64.tar.gz
```
### 修改配置文件
下面只列出已配置的参数
```bash
$ cat elasticsearch-7.0.1/config/elasticsearch.yml | egrep -v "^$|^#"
  cluster.name: efk-cluster     # 集群名称
  node.name: efk-node1          # 节点名称，212服务器改成efk-node2
  path.data: /var/lib/elasticsearch     # elasticsearch数据目录
  path.logs: /var/log/elasticsearch     # elasticsearch日志目录
  network.host: 0.0.0.0                 # elasticsearch绑定的地址
  http.port: 9200                       # elasticsearch绑定的端口
  discovery.seed_hosts: ["efk-node1", "efk-node2"]  #集群发现
  cluster.initial_master_nodes: ["efk-node1", "efk-node2"]  #第一次访问初始的集群节点
  xpack.security.enabled: true          # ssl\tls安全参数
  xpack.security.transport.ssl.enabled: true    # ssl\tls安全参数
```
### 创建运行elasticsearch用户
> elasticsearch默认是不运行root用户运行的，因此我们得创建一个新用户来运行elasticsearch。

```bash
$ useradd elastic
$ passwd elastic
```
### 创建依赖文件
```bash
$ mkdir /var/log/elasticsearch      # 创建日志目录

$ mkdir /var/lib/elasticsearch      # 创建数据目录

$ touch /var/run/elasticsearch.pid  # 创建进程文件

$ chown -R elastic.elastic /elastic # 修改elastic工作目录所有者
$ chown -R elastic.elastic /var/log/elasticsearch && \
chown -R elastic.elastic /var/lib/elasticsearch && \
chown -R elastic.elastic /var/run/elasticsearch.pid   # 相关目录所有者都修改成运行elasticsearch服务的用户
```
### 准备elasticsearch systemctl文件
```bash
[Unit]
Description=Elasticsearch
Documentation=http://www.elastic.co
Wants=network-online.target
After=network-online.target

[Service]
Environment=JAVA_HOME=/usr/local/jdk1.8.0_171
User=elastic
Group=elastic
ExecStart=/elastic/elasticsearch-7.0.1/bin/elasticsearch -p /var/run/elasticsearch.pid

LimitNOFILE=65535
LimitNPROC=65535
LimitAS=infinity
LimitFSIZE=infinity

[Install]
WantedBy=multi-user.target
```
### 启动elasticsearch
```bash
$ systemctl daemon-reload           # 加载systemctl配置文件

$ systemctl start elasticsearch     # 启动elasticsearch

$ systemctl enable elasticsearch    # 设置开机启动elasticsearch
```
>**Tips**: elasticsearch7.0.1安装完默认自带基础版的X-Pack功能，使用铂金版的需要购买或者参考[破解X-pack](https://www.ipyker.com/2019/03/13/elastic-x-pack/)进行破解。

### 验证集群状态
通过上面的配置，我们的elasticsearch服务器已经启动，并且默认监听在`9200`和`9300`端口。
`9200端口`: 为web访问提供服务；`9300端口`：为集群节点提供通信。
```bash
# 验证集群节点数，其中master为*的代表该节点为主节点
$ curl http://192.168.20.211:9200/_cat/nodes?v
ip             heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
192.168.20.212           10          39   0    0.00    0.01     0.05 mdi       -      efk-node2
192.168.20.211           18          37   0    0.00    0.01     0.05 mdi       *      efk-node1

# 验证集群健康状态，status为green表示正常
$ curl http://192.168.20.211:9200/_cat/health?v
epoch      timestamp cluster        status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1558517253 09:27:33  efk-cluster green           2         2      6   3    0    0        0             0                  -                100.0%
```
{% note success %}由上可知，我们的elasticsearch集群已经正常工作了，并且在2台集群节点下/var/lib/elasticsearch目录下都有相同的数据。更多关于elasticsearch REST API请参考官方说明。
[cluster apis](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/cluster.html) [_cat apis](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/cat.html) [Search apis](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/search.html) [Document apis](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/docs.html) {% endnote %}
## 安装 Kibana
以下操作只在`efk-node1`主机上进行：
### 下载和解压
```bash
# 进入elastic工作目录
$ cd /elastic

# 下载kibana tar包
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-7.0.1-linux-x86_64.tar.gz

# 解压到当前/elastic目录
$ tar zxvf kibana-7.0.1-linux-x86_64.tar.gz
```
### 修改配置文件
下面只列出已配置的参数
```bash
$ cat kibana-7.0.1-linux-x86_64/config/kibana.yml | egrep -v "^$|^#"
  server.port: 5601           # kibana服务端口
  server.host: "0.0.0.0"      # kibana服务地址
  elasticsearch.hosts: ["http://192.168.20.201:9200"]  # es访问地址，kibana需要和它通信
  elasticsearch.username: "elastic"            # es帐号，如果是X-pack铂金版可配
  elasticsearch.password: "pyker123456"        # es密码，如果是X-pack铂金版可配
  elasticsearch.logQueries: true               # 查询日志是否发送到ES，配合logging.json: true
  pid.file: /var/run/kibana.pid                # kibana的进行id文件
  logging.dest: /var/log/kibana.log            # kibana日志文件
  logging.json: true                           # 以json格式输出日志
  logging.verbose: true                        # 记录所有事件，包括系统使用信息和所以请求
  i18n.locale: "zh-CN"                         # 设置kibana为中文
```
### 创建依赖文件
```bash
$ touch /var/log/kibana.log      # 创建日志文件

$ touch /var/run/kibana.pid      # 创建进程文件

$ chown -R elastic.elastic /elastic && \
chown -R elastic.elastic /var/log/kibana.log && \
chown -R elastic.elastic /var/run/kibana.pid     # 修改elastic工作目录及相关工作文件所有者
```
>Tips：默认Kibana是支持root运行的，我这里为了统一elasticsearch运行环境所以打算elastic用户运行。

### 准备kibana systemctl文件
```bash
[Unit]
Description=Kibana
Documentation=http://www.elastic.co
Wants=network-online.target
After=network-online.target

[Service]
User=elastic
Group=elastic
ExecStart=/elastic/kibana-7.0.1-linux-x86_64/bin/kibana
Restart=always

[Install]
WantedBy=multi-user.target
```
### 启动kibana
```bash
$ systemctl daemon-reload           # 加载systemctl配置文件

$ systemctl start kibana            # 启动kibana

$ systemctl enable kibana           # 设置开机启动kibana
```
> Kibana默认监控在5601端口上，此时可以通过`http://192.168.20.211:5601`访问kibana。

## 安装cerebro可视化集群管理工具
cerebro是一个使用Scala，Play Framework，AngularJS和Bootstrap构建的开源（MIT许可）elasticsearch web可视化的监控工具，[Github项目](https://github.com/lmenezes/cerebro)
以下操作只在`efk-node2`主机上进行：
### 下载和解压
请前往https://github.com/lmenezes/cerebro/releases 地址下载cerebro工具
```json
$ tar zxvf cerebro-0.8.3.tgz
$ vim cerebro-0.8.3/conf/application.conf  
# 设置cerebro密码登陆认证
auth={
    type: basic
    settings: {
        username="admin"
        password="pyker123456"
    }
}
# 文件最后配置elasticsearch地址
hosts = [
  {
    host = "http://192.168.20.211:9200"
    name = "efk-cluster"
  },
```
### 准备cerebro systemctl文件
```bash
[Unit]
Description=Cerebro
After=network.target

[Service]
Environment=JAVA_HOME=/usr/local/jdk1.8.0_171
User=elastic
Group=elastic
LimitNOFILE=65535
ExecStart=/elastic/cerebro-0.8.3/bin/cerebro -Dconfig.file=/elastic/cerebro-0.8.3/conf/application.conf -Dhttp.port=1234
Restart=on-failure
WorkingDirectory=/elastic/cerebro-0.8.3

[Install]
WantedBy=multi-user.target
```
### 命令启动cerebro
```bash
# 默认9000端口
$ nohup ./bin/cerebro -Dhttp.port=1234 -Dhttp.address=192.168.20.212 &
```
### systemctl方式启动cerebro
```bash
$ systemctl daemon-reload           # 加载systemctl配置文件

$ systemctl start cerebro           # 启动cerebro

$ systemctl enable cerebro          # 设置开机启动cerebro
```
### 访问cerebro
通过浏览器访问`http://192.168.20.212:1234`就可以cerebro工具了。该工具详细的显示了es集群状态、节点数、索引数、分片数、文档数以及数据大小等参数。
![](/images/pic/cerebro.png)

## 安装filebeat
[此前文档](https://www.ipyker.com/2019/03/14/filebeat/)我们已经说明了关于filebeat的原理，以及一些配置文件参数。现在我们只初略的说明我们已使用的配置参数。以下操作只在`real-server`主机上进行：（也就是我们业务所跑的服务器，我们要抓取的是业务日志，所以当然是安装在业务服务器上咯）
### 下载和解压
```bash
# 进入elastic工作目录
$ mkdir /elastic &&& cd /elastic

# 下载kibana tar包
$ wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.0.1-linux-x86_64.tar.gz

# 解压到当前/elastic目录
$ tar zxvf filebeat-7.0.1-linux-x86_64.tar.gz
```
>{% label info@本例我们使用filebeat监控tomcat日志和nginx日志 %}
### 修改配置文件
#### 以配置文件方式支持tomcat日志输入
下面只列出已配置的参数，关于参数说明，请参考[此前文档](https://www.ipyker.com/2019/03/14/filebeat/)
```yaml
$ cat filebeat-7.0.1-linux-x86_64/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /usr/local/tomcat/logs/catalina.out   # 监控tomcat控制台catalina.out日志文件
  fields:
    log_topic: tomcat_access_logs
  exclude_lines: ['收到ping的消息']
  multiline.pattern: '^[[:space:]]+|^Caused by:'
  multiline.negate: false
  multiline.match: after
  ignore_older: 0
  close_inactive: 2m

- type: log
  enabled: true
  paths:
    - /usr/local/tomcat/logs/localhost_access.*  # 监控tomcat访问localhost_access日志文件
  fields:
    log_topic: tomcat_catalina_logs
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true
setup.template.settings:
  index.number_of_shards: 1
setup.kibana:
  host: "192.168.20.211:5601"
output.elasticsearch:     # 我们这里直接输出到ES，当然也可以输出到logstash、kafka等中间件
  hosts: ["192.168.20.211:9200"]
  #protocol: "https"
  username: "elastic"
  password: "pyker123456"
processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
```
>{% label warning@catalina.out和localhost_access都需要使用一定的格式。方便filebeat处理多行事件日志。 %}

#### 以模版方式支持nginx日志输入
默认filebeat自带诸多服务日志模版，如：nginx、redis、apache、IIS、kafka等等。默认都在filebeat解压后`module`、`modules.d`目录中。
```yaml
$ cat /elastic/filebeat-7.0.1-linux-x86_64/modules.d/nginx.yml.disabled
- module: nginx
  access:
    enabled: true
    var.paths: ["/data/wwwlogs/access.log*"]  # 配置nginx实际的访问日志路径，多个文件逗号分开
  error:
    enabled: true
    var.paths: ["/data/wwwlogs/error_nginx.log*"]  # 配置nginx实际的错误日志路径
```

#### 启动nginx模版
```bash
$ cd /elastic/filebeat-7.0.1-linux-x86_64
$ ./filebeat modules enable nginx 

$ ./filebeat modules list     # 列出已启动和未启动的模版
Enabled:
nginx
system

Disabled
apache
auditd
...
```
### 准备filebeat systemctl文件
```bash
[Unit]
Description=Filebeat sends log files to Logstash or directly to Elasticsearch.
Documentation=https://www.elastic.co/products/beats/filebeat
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/elastic/filebeat-7.0.1-linux-x86_64/filebeat -c /elastic/filebeat-7.0.1-linux-x86_64/filebeat.yml
Restart=always

[Install]
WantedBy=multi-user.target
```
### 启动filebeat
```bash
$ systemctl daemon-reload           # 加载systemctl配置文件

$ systemctl start filebeat            # 启动filebeat

$ systemctl enable filebeat           # 设置开机启动filebeat
```
至此简单的EFK集群搭建完成。在生成环境中 我们还会在filebeat和elasticsearch中间加入kafka和logstash来提高日志数据的高效传输和更强的日志过滤功能，而kafka又可以配置成集群模式。

**在当前文档中我们并未说明kibana如何使用，请参考[官方使用教程](https://www.elastic.co/guide/cn/kibana/current/introduction.html)**