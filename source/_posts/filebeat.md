layout: post
title: Elastic Stack之Filebeat7.0详解
author: Pyker
categories: elastic
img: /images/bar/beats.png
tags:
  - filebeat
  - efk
top: false
cover: false
date: 2019-03-14 12:10:00
---
## 一：认识Filebeat
* `Filebeat`是beats其中一个组件，它是一个`轻量型日志采集器`，可以直接（或者通过Logstash）将数据发送到Elasticsearch，在那里你可以进一步处理和增强数据，然后在Kibana中将其可视化。

* 当您要面对成百上千、甚至成千上万的服务器、虚拟机和容器生成的日志时，请告别 SSH 吧。Filebeat 将为您提供一种轻量型方法，用于转发和汇总日志与文件，让简单的事情不再繁杂。

* 它性能稳健，不错过任何检测信号，无论在任何环境中，随时都潜伏着应用程序中断的风险。Filebeat 能够读取并转发日志行，如果出现中断，还会在一切恢复正常后，从中断前停止的位置继续开始。

* `Filebeat` 内置有多种模块（auditd、Apache、NGINX、System、MySQL 等等），可针对常见格式的日志大大简化收集、解析和可视化过程，只需一条命令即可。之所以能实现这一点，是因为它将自动默认路径（因操作系统而异）与 Elasticsearch 采集节点管道的定义和 Kibana 仪表板组合在一起。不仅如此，数个 Filebeat 模块还包括预配置的 Machine Learning 任务。

## 二：Filebeat工作原理
Filebeat由两个主要组件组成：`inputs` 和 `harvesters`（直译：收割机，采集器）。这些组件一起工作以跟踪文件，并将事件数据发送到你指定的输出。
### 2.1 harvester是什么
一个`harvester`负责读取一个单个文件的内容。harvester逐行读取每个文件（一行一行地读取每个文件），并把这些内容发送到输出。每个文件启动一个harvester负责打开和关闭这个文件，这就意味着在harvester运行时文件描述符保持打开状态。而在harvester正在读取文件内容的时候，文件被删除或者重命名了，那么Filebeat会继续读这个文件。这就有一个问题了，就是只要负责这个文件的harvester没用关闭，那么磁盘空间就不会释放。默认情况下，Filebeat保存文件打开直到close_inactive到达。
### 2.2 input是什么
一个`input`负责管理当前的harvesters，并找到所有要读取的源。如果input类型是log，则input查找paths已定义的glob路径匹配的所有日志文件，并为每个文件启动一个harvester。
下面的例子配置Filebeat从所有匹配指定的glob模式的文件中读取行：
```yaml
filebeat.inputs:
- type: log
  paths:
    - /var/log/*.log
    - /var/path2/*.log
```
### 2.3 Filebeat如何保持文件状态
Filebeat保存每个文件的状态，并经常刷新状态到磁盘上的注册文件（registry）。状态用于记住harvester读取文件的最后一个偏移量（行），并确保所有日志行被发送到输出（如Elasticsearch 或者 Logstash等），当输出无法访问时，那么Filebeat会跟踪已经发送的最后一行，并只要输出再次变得可用时继续读取文件。当Filebeat运行时，会将每个文件的状态新保存在内存中。当Filebeat重新启动时，将使用注册文件中的数据重新构建状态，Filebeat将在最后一个已知行位置继续每个harvester。对于每个输入，Filebeat保存它找到的每个文件的状态。因为文件可以重命名或移动，所以文件名和路径不足以标识文件。对于每个文件，Filebeat存储惟一标识符，以检测文件是否以前读取过。
如果你的情况涉及每天创建大量的新文件，你可能会发现注册表文件变得太大了。为了减小注册表文件的大小，有两个配置选项可用：`clean_remove`和`clean_inactive`。对于你不再访问且被忽略的旧文件，建议您使用`clean_inactive`。如果想从磁盘上删除旧文件，那么使用`clean_remove`选项。
### 2.4 Filebeat如何确保至少投递一次（at-least-once）
Filebeat保证事件将被投递到配置的输出中至少一次，并且不会丢失数据。Filebeat能够实现这种行为，因为它将每个事件的投递状态存储在注册表文件中。
在定义的输出被阻塞且没有确认所有事件的情况下，Filebeat将继续尝试发送事件，直到输出确认收到事件为止。
如果Filebeat在发送事件的过程中关闭了，则在关闭之前它不会等待输出确认所有事件。当Filebeat重新启动时，发送到输出（但在Filebeat关闭前未确认）的任何事件将再次发送。这确保每个事件至少被发送一次，但是你最终可能会将重复的事件发送到输出。你可以通过设置`shutdown_timeout`选项，将Filebeat配置为在关闭之前等待特定的时间。
## 三：配置Filebeat
为了配置Filebeat，你可以编辑配置文件`filebeat.yml`。此外同级目录下还有一个完整的配置文件示例`filebeat.reference.yml`
### 3.1 指定运行哪个模块
Filebeat提供了几种启用模块的不同方式：
* 用`modules.d`目录下的配置启用模块
* 运行Filebea命令的时候启用模块
* 配置`filebeat.yml`文件启用模块配置

```bash
# 用modules.d目录启用模块配置
$ ./filebeat modules enable apache2 mysql

# 运行Filebea命令的时候启用模块
$ ./filebeat --modules nginx,mysql,system

# 配置`filebeat.yml`文件启用模块配置
filebeat.modules:
 - module: nginx
 - module: mysql
 - module: system
```
### 3.2 配置inputs
为了手动配置Filebeat（代替用模块），你可以在filebeat.yml中的`filebeat.inputs`区域下指定一个inputs列表。列表是一个YMAL数组，并且你还可以指定多个inputs，相同input类型也可以指定多个。例如：
```yaml
filebeat.inputs:
- type: log
  paths:
    - /var/log/system.log
    - /var/log/wifi.log
- type: log
  paths:
    - "/var/log/apache2/*"
  fields:
    apache: true
  fields_under_root: true
```
上面案例是从日志文件中读取行，类型为`log`的input需要指定一个paths列表，列表中的每一项必须能够定位并抓取到日志行。此外你还可以配置其它额外的配置项（比如，fields, include_lines, exclude_lines, multiline等等）来从这些文件中过滤读取行。你设置的这些配置对当前所有这种类型的input在获取日志行的时候都生效。为了对不同的文件应用不同的配置，你需要定义多个input区域。
### 3.3 其他配置项
* `paths`：从预定义的目录级别下抓取所有文件。
例如：/var/log/*/*.log 将会抓取/var/log子目录目录下所有.log文件。它不会抓取/var/log本身目录下的日志文件。也不会抓取子目录的子目录下的日志文件。如果你应用`recursive_glob: true`设置的话，它将递归地抓取所有子目录下的所有.log文件。

* `recursive_glob.enabled`
允许将\*\*扩展为递归glob模式。启用这个特性后，每个路径中最右边的\*\*被扩展为固定数量的glob模式。例如：`/foo/**`扩展到`/foo`， `/foo/*`， `/foo/**`，等等。如果启用，它将单个\*\*扩展为8级深度\*模式。这个特性默认是启用的，设置`recursive_glob.enabled: false`可以禁用它。

* `encoding`
按照[W3C推荐的HTML5](http://www.w3.org/TR/encoding)读取的文件的编码。例如：`plain`, `latin1`, `utf-8`, `utf-16be-bom`, `utf-16be`, `utf-16le`, `big5`, `gb18030`, `gbk`, `hz-gb-2312`。`plain`编码是特殊的，因为它不校验或者转换任何输入。

* `exclude_lines`
一组正则表达式，用于匹配你想要排除的行。Filebeat会删除这组正则表达式匹配的行。默认情况下，没有行被删除。空行被忽略。如果指定了`multiline`，那么在用`exclude_lines`过滤之前会将每个多行消息合并成一个单行。（PS：也就是说，多行合并成单行后再支持排除行的过滤）
例如配置Filebeat删除以DBG开头的行：
    ```yaml
    filebeat.inputs:
    - type: log
      ...
      exclude_lines: ['^DBG']
    ```
* `include_lines`
一组正则表达式，用于匹配你想要包含的行。Filebeat只会导出匹配这组正则表达式的行。默认情况下，所有行都被导出。空行被忽略。如果指定了`multipline`设置，每个多行消息先被合并成单行后再执行`include_lines`过滤。
例如配置Filebeat`导出以ERR或者WARN开头`的行：
    ```yaml
    filebeat.inputs:
    - type: log
      ...
      include_lines: ['^ERR', '^WARN']
    ```
>如果`include_lines` 和 `exclude_lines` 都被定义了，那么Filebeat先执行include_lines后执行exclude_lines，而与这两个选项被定义的顺序没有关系。
例如导出那些除了以DGB开头的所有包含sometext的行：
    ```yaml
    filebeat.inputs:
    - type: log
      ...
      include_lines: ['sometext']
      exclude_lines: ['^DBG']
    ```
* `exclude_files`
需要排除的日志文件。 要匹配的正则表达式列表。 Filebeat从列表中删除与任何正则表达式匹配的文件。 默认情况下，不会删除任何文件。
例如忽略.gz的文件:
    ```yaml
    filebeat.inputs:
    - type: log
      ...
      exclude_files: ['\.gz$']
    ```


* `fields`
可选的附加字段。 可以自由选择这些字段，以便将其他信息添加到已抓取到的日志文件中进行过滤。

* `fields_under_root`
设置为true可将其他字段存储为顶级字段，而不是“字段”子字典下。 如果名称与Filebeat本身添加的字段冲突，则自定义字段会覆盖默认字段。

* `harvester_buffer_size`
当抓取一个文件时每个harvester使用的buffer的字节数。默认是16384。

* `max_bytes`
单个日志消息允许的最大字节数。超过max_bytes的字节将被丢弃且不会发送到输出。对于多行日志消息来说这个设置是很有用的，因为它们往往很大。默认是10MB（10485760）。

* `json`
    `json.keys_under_root`
    默认情况下，解码后的JSON被放置在一个以"json"为key的输出文档中。如果你启用这个设置，那么这个key在文档中被复制为顶级。默认是false。

    `json.overwrite_keys`
    如果启用了keys_under_root和该设置，则解码的JSON对象中的值将覆盖Filebeat通常添加的字段（类型，来源，偏移等），以防发生冲突。

    `json.add_error_key`
    如果启用次设置，则当JSON解析出现错误的时候Filebeat添加 `error.message`和 `error.type: json`两个key，或者当没有使用message_key的时候。

    `json.message_key`
    一个可选的配置，用于在应用行过滤和多行设置的时候指定一个JSON key。指定的这个key必须在JSON对象中是顶级的，而且其关联的值必须是一个字符串，否则没有过滤或者多行聚集发送。

    `ignore_decoding_error`
    一个可选的配置，用于指定是否JSON解码错误应该被记录到日志中。如果设为true，错误将被记录。默认是false。
    ```yaml
    json.keys_under_root: true
    json.add_error_key: true
    json.message_key: log
    ```
* `multiline`
    `multiline.pattern`
    指定要匹配的正则表达式模式。根据您配置其他多行选项的方式，与指定正则表达式匹配的行被视为前一行的延续或新多行事件的开始。 您可以设置negate选项来否定模式。
    `multiline.negate`
    定义模式是否被否定。 默认值为false。
    `multiline.match`
    指定Filebeat如何将匹配行组合到事件中。 设置是在之前或之后。 这些设置的行为取决于您为negate指定的内容。
    ```yaml
    multiline.pattern: '^\['
    multiline.negate: true
    multiline.match: after
    ```

multiline.negate | multiline.match | Result
--- | --- | ---
false | after | 与模式匹配的连续行和前面不匹配的行合并成一条完整日志
false | before | 与模式匹配的连续行和后面不匹配的行合并成一条完整日志
true | after | 与模式匹配的行和后面不匹配的行组成一条完整日志
true | before | 与模式匹配的行和前面不匹配的行组成一条完整日志
![官方说明](/images/pic/multiline.jpg)

例如Java堆栈跟踪由多行组成，在初始行之后的每一行都以空格开头，例如下面这样：
```
Exception in thread "main" java.lang.NullPointerException
        at com.example.myproject.Book.getTitle(Book.java:16)
        at com.example.myproject.Author.getBookTitles(Author.java:25)
        at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
```
为了把这些行合并成单个事件，用写了多行配置：
```yaml
multiline.pattern: '^[[:space:]]'
multiline.negate: false
multiline.match: after
```
下面是一个稍微更复杂的例子
```
Exception in thread "main" java.lang.IllegalStateException: A book has a null property
       at com.example.myproject.Author.getBookIds(Author.java:38)
       at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
Caused by: java.lang.NullPointerException
       at com.example.myproject.Book.getId(Book.java:22)
       at com.example.myproject.Author.getBookIds(Author.java:35)
       ... 1 more
```
为了合并这个，用下面的配置：
```yaml
multiline.pattern: '^[[:space:]]+(at|\.{3})\b|^Caused by:'
multiline.negate: false
multiline.match: after
```
在这个例子中，模式匹配下列行：
1. 以空格开头，后面跟 at 或者 ... 的行
2. 以 Caused by: 开头的行

* `ignore_older`
如果启用，那么Filebeat会忽略在指定的时间跨度之前被修改的文件。如果你想要保留日志文件一个较长的时间，那么配置`ignore_older`是很有用的。例如，如果你想要开始Filebeat，但是你只想发送最近一周最新的文件，这个情况下你可以配置这个选项。你可以用时间字符串，比如2h（2小时），5m（5分钟）。默认是0，意思是禁用这个设置。你必须设置`ignore_older`比`close_inactive`更大。

* `close_*`
close_*配置项用于在一个确定的条件或者时间点之后`关闭harvester`。关闭harvester意味着关闭文件处理器。如果在harvester关闭以后文件被更新，那么在`scan_frequency`结束后改文件将再次被拾起。然而，当harvester关闭的时候如果文件被删除或者被移动，那么Filebeat将不会被再次拾起，并且这个harvester还没有读取的数据将会丢失。

    `close_inactive`
    当启用此选项时，如果文件在指定的持续时间内未被获取，则Filebeat将关闭文件句柄。当harvester读取最后一行日志时，指定周期的计数器就开始工作了。它不基于文件的修改时间。如果关闭的文件再次更改，则会启动一个新的harvester，并且在`scan_frequency`结束后，将获得最新的更改。
    推荐给`close_inactive`设置一个比你的日志文件更新频率更大一点儿的值。例如，如果你的日志文件每隔几秒就会更新，你可以设置`close_inactive`为1m。如果日志文件的更新速率不固定，那么可以用多个配置。将`close_inactive`设置为更低的值意味着文件句柄可以更早关闭。然而，这样做的副作用是，如果harvester关闭了，新的日志行不会实时发送。
    关闭文件的时间戳不依赖于文件的修改时间。代替的，Filebeat用一个内部时间戳来反映最后一次读取文件的时间。例如，如果`close_inactive`被设置为5分钟，那么在harvester读取文件的最后一行以后，这个5分钟的倒计时就开始了。你可以用时间字符串，比如2h（2小时），5m（5分钟）。默认是5m。

    `close_renamed`
    当启用此选项时，Filebeat会在重命名文件时关闭文件处理器。默认情况下，harvester保持打开状态并继续读取文件，因为文件处理器不依赖于文件名。如果启用了`close_rename`选项，并且重命名或者移动的文件不再匹配文件模式的话，那么文件将不会再次被选中。Filebeat将无法完成文件的读取。

    `close_removed`
    当启用此选项时，Filebeat会在删除文件时关闭harvester。通常，一个文件只有在它在由`close_inactive`指定的期间内不活跃的情况下才会被删除。但是，如果一个文件被提前删除，并且你不启用`close_removed`，则Filebeat将保持文件打开，以确保harvester已经完成。如果由于文件过早地从磁盘中删除而导致文件不能完全读取，请禁用此选项。

    `close_eof`
    抓取到达文件末尾后立即关闭文件处理程序。默认情况下，此选项被禁用。 注意：潜在的数据丢失。 请务必阅读并理解此选项的文档。

    `close_timeout` 
    当启用此选项是，Filebeat会给每个harvester一个预定义的生命时间。无论读到文件的什么位置，只要`close_timeout`周期到了以后就会停止读取。当你想要在文件上只花费预定义的时间时，这个选项对旧的日志文件很有用。尽管在`close_timeout`时间以后文件就关闭了，但如果文件仍然在更新，则Filebeat将根据已定义的`scan_frequency`再次启动一个新的harvester。这个harvester的`close_timeout`将再次启动，为超时倒计时。

* `scan_frequency`
Filebeat多久检查一次指定路径下的新文件（PS：检查的频率）。例如，如果你指定的路径是 /var/log/* ，那么会以指定的`scan_frequency`频率去扫描目录下的文件（PS：周期性扫描）。指定1秒钟扫描一次目录，默认值10秒。如果你需要近实时的发送日志行的话，不要设置`scan_frequency`为一个很低的值，而应该调整`close_inactive`以至于文件处理器保持打开状态，并不断地轮询你的文件。

更多关于类型为`log`的输入配置，请参考[官方配置文档](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html)
## 四：加载外部配置文件
Filebeat可以为输入和模块加载外部配置文件，允许您将配置分成多个较小的配置文件。 
### 4.1 配置input
对于输入配置，请在filebeat.yml文件的filebeat.config.inputs部分中指定path选项。 例如：
```yaml
filebeat.config.inputs:
  enabled: true
  path: configs/*.yml
```
每一个在path下的文件都必须包含一个或多个input定义，例如：
```yaml
- type: log
  paths:
    - /var/log/mysql.log
  scan_frequency: 10s

- type: log
  paths:
    - /var/log/apache.log
  scan_frequency: 5s
```
### 4.2 模块配置
对于模块配置，请在filebeat.yml文件的filebeat.config.modules部分中指定path选项。 默认情况下，Filebeat会加载modules.d目录中启用的模块配置。 例如：
```yaml
filebeat.config.modules:
  enabled: true
  path: ${path.config}/modules.d/*.yml
```
每个被发现的配置文件必须包含一个或多个模块定义，例如：
```
- module: apache2
  access:
    enabled: true
    var.paths: [/var/log/apache2/access.log*]
  error:
    enabled: true
    var.paths: [/var/log/apache2/error.log*]
```
### 4.3 配置output
您可以通过在filebeat.yml配置文件的输出部分中设置选项来配置Filebeat以写入特定输出。 只能定义一个输出。支持的输出如：`Elasticsearch`, `Logstash`, `Kafka`, `Redis`, `File`, `Console`, `Cloud`。
#### 4.3.1 配置Elasticsearch输出
为输出指定Elasticsearch时，Filebeat会使用Elasticsearch HTTP API将事务直接发送到Elasticsearch。例如：
```yaml
output.elasticsearch:
  hosts: ["https://localhost:9200"]
  index: "filebeat-%{[beat.version]}-%{+yyyy.MM.dd}"
  ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]
  ssl.certificate: "/etc/pki/client/cert.pem"
  ssl.key: "/etc/pki/client/cert.key"
```
为了启用SSL，只需要在hosts下的所有URL添加https即可
```yaml
output.elasticsearch:
  hosts: ["https://localhost:9200"]
  username: "filebeat_internal"
  password: "YOUR_PASSWORD"
```
如果Elasticsearch节点是用IP:PORT的形式定义的，那么添加protocol:https。
```yaml
output.elasticsearch:
  hosts: ["localhost"]
  protocol: "https"
  username: "{beatname_lc}_internal"
  password: "{pwd}"
```
#### 4.3.2 Elasticsearch输出配置项
`enabled`
启用或禁用该输出。默认true。

`hosts`
Elasticsearch节点列表。事件以循环顺序发送到这些节点。如果一个节点变得不可访问，那么自动发送到下一个节点。每个节点可以是URL形式，也可以是IP:PORT形式。如果端口没有指定，用9200。
```yaml
output.elasticsearch:
  hosts: ["10.45.3.2:9220", "10.45.3.1:9230"]
  protocol: https
  path: /elasticsearch
```
`username`
用于认证的用户名

`password`
用户认证的密码

`protocol`
可选值是：http 或者 https。默认是http。

`path`
HTTP API调用前的HTTP路径前缀。这对于Elasticsearch监听HTTP反向代理的情况很有用。

`headers`
将自定义HTTP头添加到Elasticsearch输出的每个请求。

`index`
索引名字。（PS：意思是要发到哪个索引中去）。默认是`"filebeat-%{[beat.version]}-%{+yyyy.MM.dd}"`（例如，"filebeat-6.3.2-2017.04.26"）。如果你想改变这个设置，你需要配置 `setup.template.name` 和 `setup.template.pattern` 选项。如果你用内置的Kibana dashboards，你也需要设置`setup.dashboards.index`选项。

`indices`
索引选择器规则数组，支持条件、基于格式字符串的字段访问和名称映射。如果索引缺失或没有匹配规则，将使用index字段。例如：
```yaml
output.elasticsearch:
  hosts: ["http://localhost:9200"]
  index: "logs-%{[beat.version]}-%{+yyyy.MM.dd}"
  indices:
    - index: "critical-%{[beat.version]}-%{+yyyy.MM.dd}"
      when.contains:
        message: "CRITICAL"
    - index: "error-%{[beat.version]}-%{+yyyy.MM.dd}"
      when.contains:
        message: "ERR"
```
`timeout`
请求超时时间。默认90秒。

更多关于elasticsearch输出配置请参考[官方文档](https://www.elastic.co/guide/en/beats/filebeat/current/elasticsearch-output.html#_configuration_options_8)
这里只以elasticsearch输出类型为例，更多关于output 输出类型配置，请参考官方[配置输出类型](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-output.html)
### 4.4 加载索引模板
在filebeat.yml配置文件的`setup.template`区域指定索引模板，用来设置在Elasticsearch中的映射。如果模板加载是启用的（默认的），Filebeat在成功连接到Elasticsearch后自动加载索引模板。你可以调整下列设置或者覆盖一个已经存在的模板。

`setup.template.enabled`
设为false表示禁用模板加载

`setup.template.name`
模板的名字。默认是filebeat。Filebeat的版本总是跟在名字后面，所以最终的名字是 filebeat-%{[beat.version]}

`setup.template.pattern`
模板的模式。默认模式是filebeat-*。例如：
```yaml
setup.template.name: "filebeat"
setup.template.pattern: "filebeat-*"
```
`setup.template.fields`
描述字段的YAML文件路径。默认是 fields.yml。

`setup.template.overwrite`
是否覆盖存在的模板。默认false。

`setup.template.settings._source`
```yaml
setup.template.name: "filebeat"
setup.template.fields: "fields.yml"
setup.template.overwrite: false
setup.template.settings:
  _source.enabled: false
```
有关filebeat完整的配置文档，请参考(https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-getting-started.html)