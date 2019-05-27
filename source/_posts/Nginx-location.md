layout: post
title: 详解nginx之location误区
author: Pyker
categories: web
img: /images/bar/nginx.jpg
tags:
  - nginx
top: true
cover: false
date: 2018-01-23 18:47:00
---
### location 的匹配顺序是“先匹配正则，再匹配普通”。
矫正： location 的匹配顺序其实是“先匹配普通，再匹配正则”。我这么说，大家一定会反驳我，因为按“先匹配普通，再匹配正则”解释不了大家平时习惯的按“先匹配正则，再匹配普通”的实践经验。这里我只能暂时解释下，造成这种误解的原因是：正则匹配会覆盖普通匹配（实际的规则，比这复杂，后面会详细解释）。
###  location 的执行逻辑跟 location 的编辑顺序无关。
矫正：这句话不全对，“普通 location ”的匹配规则是“最大前缀”，因此“普通 location ”的确与 location 编辑顺序无关；但是“正则 location ”的匹配规则是“顺序匹配，且只要匹配到第一个就停止后面的匹配”；“普通location ”与“正则 location ”之间的匹配顺序是？先匹配普通 location ，再“考虑”匹配正则 location 。注意这里的“考虑”是“可能”的意思，也就是说匹配完“普通 location ”后，有的时候需要继续匹配“正则 location ”，有的时候则不需要继续匹配“正则 location ”。两种情况下，不需要继续匹配正则 location ：（ 1 ）当普通 location 前面指定了“ ^~ ”，特别告诉 Nginx 本条普通 location 一旦匹配上，则不需要继续正则匹配；（ 2 ）当普通location 恰好严格匹配上，不是最大前缀匹配，则不再继续匹配正则。

**总结一句话：  “正则 location 匹配让步普通 location 的严格精确匹配结果；但覆盖普通 location 的最大前缀匹配结果”**
>匹配优先级为： (location =) > (location 完整路径) > (location ^~ 路径) > (location ~,~* 正则顺序) > (location 部分起始路径) > (/)

**官方文档解释**：<http://wiki.nginx.org/NginxHttpCoreModule#location>
```
location
syntax: `location [=|~|~*|^~|@] /uri/ { … }`
default: no
context: server
```
>This directive allows different configurations depending on the URI.

（译者注：1 、different configurations depending on the URI 说的就是语法格式：`location [=|~|~*|^~|@] /uri/ { … }` ，依据不同的前缀`“= ”，“^~ ”，“~ ”，“~* ”`和不带任何前缀的（因为[A] 表示可选，可以不要的），表达不同的含义, 简单的说尽管location 的/uri/ 配置一样，但前缀不一样，表达的是不同的指令含义。2 、查询字符串不在URI范围内。例如：/films.htm?fid=123 的URI 是/films.htm 。）
It can be configured using both literal strings and regular expressions. To use regular expressions, you must use a prefix:
`“~”`for case sensitive matching
`“~*”`for case insensitive matching
译文：上文讲到`location /uri/` 可通过使用不同的前缀，表达不同的含义。对这些不同前缀，分下类，就2 大类：正则location ，英文说法是location using regular expressions 和普通location ，英文说法是location using literal strings 。那么其中“~ ”和“~* ”前缀表示正则location ，“~ ”区分大小写，“~* ”不区分大小写；其他前缀（包括：“=”，“^~ ”和“@ ”）和无任何前缀的都属于普通location 。

>To determine which location directive matches a particular query, the literal strings are checked first.

译文：对于一个特定的 HTTP 请求（ a particular query ）， nginx 应该匹配哪个 location 块的指令呢（注意：我们在 nginx.conf 配置文件里面一般会定义多个 location 的）？匹配 规则是：先匹配普通location （再匹配正则表达式）。注意：官方文档这句话就明确说了，先普通location ，而不是有些同学的误区“先匹配正则location ”。
>Literal strings match the beginning portion of the query – the most specific match will be used.

前面说了“普通location ”与“正则location ”之间的匹配规则是：先匹配普通location ，再匹配正则location 。那么，“普通location ”内部（普通location 与普通location ）是如何匹配的呢？简单的说：最大前缀匹配。原文：1、match the beginning portion of the query （说的是匹配URI 的前缀部分beginning portion ）； 2 、the most specific match will be used （因为location 不是“严格匹配”，而是“前缀匹配”，就会产生一个HTTP 请求，可以“前缀匹配”到多个普通location ，例如：``location /prefix/mid/ {}`` 和`location /prefix/ {}` ，对于HTTP 请求/prefix/mid/t.html ，前缀匹配的话两个location 都满足，选哪个？原则是：the most specific match ，于是选的是`location /prefix/mid/ {}` ）。
>Afterwards, regular expressions are checked in the order defined in the configuration file. The first regular expression to match the query will stop the search.

这段话说了两层意思，第一层是：“Afterwards, regular expressions are checked ”, 意思是普通location 先匹配，而且选择了最大前缀匹配后，就不能停止后面的匹配，最大前缀匹配只是一个临时的结果，nginx 还需要继续检查正则location （但至于最终是和普通location 的最大前缀匹配，还是正则location 的匹配，截止当前的内容还没讲，但后面会讲）。第二层是“regular expressions are checked in the order defined in the configuration file. The first regular expression to match the query will stop the search. ”，意思是说“正则location ”与“正则location”内部的匹配规则是：按照正则location 在配置文件中的物理顺序（编辑顺序）匹配的（这句话就说明location 并不是一定跟顺序无关，只是普通location 与顺序无关，正则location 还是与顺序有关的），并且只要匹配到一条正则location ，就不再考虑后面的（这与“普通location ”与“正则location ”之间的规则不一样，“普通location ”与“正则location ”之间的规则是：选择出“普通location ”的最大前缀匹配结果后，还需要继续搜索正则location ）。
>If no regular expression matches are found, the result from the literal string search is used.

这句话回答了“普通location ”的最大前缀匹配结果与继续搜索的“正则location ”匹配结果的决策关系。如果继续搜索的“正则location ”也有匹配上的，那么“正则location ”覆盖 “普通location ”的最大前缀匹配（因为有这个覆盖关系，所以造成有些同学以为正则location 先于普通location 执行的错误理解）；但是如果“正则location ”没有能匹配上，那么就用“普通location ”的最大前缀匹配结果。
For case insensitive operating systems, like Mac OS X or Windows with Cygwin, literal string matching is done in a case insensitive way (0.7.7). However, comparison is limited to single-byte locale’s only.
Regular expression may contain captures (0.7.40), which can then be used in other directives.

>It is possible to disable regular expression checks after literal string matching by using “^~” prefix.If the most specific match literal location has this prefix: regular expressions aren’t checked.

通常的规则是，匹配完了“普通location ”指令，还需要继续匹配“正则location ”，但是你也可以告诉Nginx ：匹配到了“普通location ”后，不再需要继续匹配“正则location ”了，要做到这一点只要在“普通location ”前面加上“^~ ”符号（^ 表示“非”，~ 表示“正则”，字符意思是：不要继续匹配正则）。
>By using the “=” prefix we define the exact match between request URI and location. When matched search stops immediately. E.g., if the request “/” occurs frequently, using “location = /” will speed up processing of this request a bit as search will stop after first comparison.

除了上文的“^~ ”可以阻止继续搜索正则location 外，你还可以加“= ”。那么如果“^~ ”和“= ”都能阻止继续搜索正则location 的话，那它们之间有什么区别呢？区别很简单，共同点是它们都能阻止继续搜索正则location ，不同点是“^~ ”依然遵守“最大前缀”匹配规则，然而“= ”不是“最大前缀”，而是必须是严格匹配（exact match ）。
这里顺便讲下`“location / {} ”和“location = / {} ”`的区别，“location / {} ”遵守普通location 的最大前缀匹配，由于任何URI 都必然以“/ ”根开头，所以对于一个URI ，如果有更specific 的匹配，那自然是选这个更specific 的，如果没有，“/ ”一定能为这个URI 垫背（至少能匹配到“/ ”），也就是说“location / {} ”有点默认配置的味道，其他更specific的配置能覆盖overwrite 这个默认配置（这也是为什么我们总能看到location / {} 这个配置的一个很重要的原因）。而“location = / {} ”遵守的是“严格精确匹配exact match ”，也就是只能匹配 http://host:port/ 请求，同时会禁止继续搜索正则location 。因此如果我们只想对“GET / ”请求配置作用指令，那么我们可以选“location = / {} ”这样能减少正则location 的搜索，因此效率比“location / {}” 高（注：前提是我们的目的仅仅只想对“GET / ”起作用）。
>On exact match with literal location without “=” or “^~” prefixes search is also immediately terminated.

前面我们说了，普通location 匹配完后，还会继续匹配正则location ；但是nginx 允许你阻止这种行为，方法很简单，只需要在普通location 前加“^~ ”或“= ”。但其实还有一种“隐含”的方式来阻止正则location 的搜索，这种隐含的方式就是：当“最大前缀”匹配恰好就是一个“严格精确（exact match ）”匹配，照样会停止后面的搜索。原文字面意思是：只要遇到“精确匹配exact match ”，即使普通location 没有带“= ”或“^~ ”前缀，也一样会终止后面的匹配。

先举例解释下，后面例题会用实践告诉大家。假设当前配置是：location /exact/match/test.html { 配置指令块1}，location /prefix/ { 配置指令块2} 和 location ~ \.html$ { 配置指令块3} ，如果我们请求 GET /prefix/index.html ，则会被匹配到指令块3 ，因为普通location /prefix/ 依据最大匹配原则能匹配当前请求，但是会被后面的正则location 覆盖；当请求GET /exact/match/test.html ，会匹配到指令块1 ，因为这个是普通location 的exact match ，会禁止继续搜索正则location 。

**To summarize, the order in which directives are checked is as follows:**
1. Directives with the “=” prefix that match the query exactly. If found, searching stops.
2. All remaining directives with conventional strings. If this match used the “^~” prefix, searching stops.
3. Regular expressions, in the order they are defined in the configuration file.
4. If #3 yielded a match, that result is used. Otherwise, the match from #2 is used.

这个顺序没必要再过多解释了。但我想用自己的话概括下上面的意思“正则 location 匹配让步普通location 的严格精确匹配结果；但覆盖普通 location 的最大前缀匹配结果”。
>It is important to know that nginx does the comparison against decoded URIs. For example, if you wish to match “/images/ /test”, then you must use “/images/ /test” to determine the location.

在浏览器上显示的URL 一般都会进行URLEncode ，例如“空格”会被编码为  ，但是Nginx 的URL 的匹配都是针对URLDecode 之后的。也就是说，如果你要匹配“/images/ /test ”，你写location 的时候匹配目标应该是：“/images/ /test ”。

**Example:**
```
location   = / {
 # matches the query / only.
 [ configuration A ]
}

location   / {
  # matches any query, since all queries begin with /, but regular
  # expressions and any longer conventional blocks will be
  # matched first.
 [ configuration B ]
}

location ^~ /images/ {
    # matches any query beginning with /images/ and halts searching,
    # so regular expressions will not be checked.
 [ configuration C ]
}

location ~* \.(gif|jpg|jpeg)$ {
 # matches any request ending in gif, jpg, or jpeg. However, all
 # requests to the /images/ directory will be handled by
 # Configuration C.  
 [ configuration D ]
}
```
上述这4 个location 的配置，没什么好解释的，唯一需要说明的是location / {[configuration B]} ，原文的注释严格来说是错误的，但我相信原文作者是了解规则的，只是文字描述上简化了下，但这个简化容易给读者造成“误解：先检查正则location ，再检查普通location ”。原文：“matches any query, since all queries begin with /, butregular expressions and any longer conventional blocks will be matched first. ”大意是说：“location / {} 能够匹配所有HTTP 请求，因为任何HTTP 请求都必然是以‘/ ’开始的（这半句没有错误）。但是，正则location 和其他任何比‘/ ’更长的普通location （location / {} 是普通location 里面最短的，因此其他任何普通location 都会比它更长，当然location = / {} 和 location ^~ / {} 是一样长的）会优先匹配（matched first ）。” 原文作者说“ but regular expressions will be matched first. ”应该只是想说正则 location 会覆盖这里的 location / {} ，但依然是普通location / {} 先于正则 location 匹配，接着再正则 location 匹配；但其他更长的普通 location （ any longer conventional blocks ）的确会先于 location / {} 匹配。

Example requests:

* / -> configuration A
* /documents/document.html -> configuration B
* /images/1.gif -> configuration C
* /documents/1.jpg -> configuration D

>Note that you could define these 4 configurations in any order and the results would remain the same.

需要提醒下：这里说“in any order ”和“… remain the same ”是因为上面只有一个正则location 。文章前面已经说了正则location 的匹配是跟编辑顺序有关系的。
>While nested locations are allowed by the configuration file parser, their use is discouraged and may produce unexpected results.

实际上 nginx 的配置文件解析程序是允许 location 嵌套定义的（ location / { location /uri/ {} } ）。但是我们平时却很少看见这样的配置，那是因为 nginx 官方并不建议大家这么做，因为这样会导致很多意想不到的后果。
>The prefix “@” specifies a named location. Such locations are not used during normal processing of requests, they are intended only to process internally redirected requests (see error_page ,try_files ).

文章开始说了location 的语法中，可以有“= ”，“^~ ”，“~ ”和“~* ”前缀，或者干脆没有任何前缀，还有“@ ”前缀，但是后面的分析我们始终没有谈到“@ ”前缀。文章最后点内容，介绍了“＠”的用途：“@ ”是用来定义“Named Location ”的（你可以理解为独立于“普通location （location using literal strings ）”和“正则location （location using regular expressions ）”之外的第三种类型），这种“Named Location ”不是用来处理普通的HTTP 请求的，它是专门用来处理“内部重定向（internally redirected ）”请求的。注意：这里说的“内部重定向（internally redirected ）”或许说成“forward ”会好点，以为内internally redirected 是不需要跟浏览器交互的，纯粹是服务端的一个转发行为。

### location 实例练习
Nginx 的语法形式是：`location [=|~|~*|^~|@] /uri/ { … } `，意思是可以以“ = ”或“ ~* ”或“ ~ ”或“ ^~ ”或“ @ ”符号为前缀，当然也可以没有前缀（因为 [A] 是表示可选的 A ； A|B 表示 A 和 B 选一个），紧接着是 /uri/ ，再接着是{…} 指令块，整个意思是对于满足这样条件的 /uri/ 适用指令块 {…} 的指令。

上述各种 location 可分两大类，分别是：“普通 location ”，官方英文说法是 location using   literal strings 和“正则 location ”，英文说法是 location using regular expressions 。其中“普通 location ”是以“ = ”或“ ^~ ”为前缀或者没有任何前缀的 /uri/ ；“正则 location ”是以“ ~ ”或“ ~* ”为前缀的 /uri/ 。
那么，当我们在一个 server 上下文编写了多个 location 的时候， Nginx 对于一个 HTTP 请求，是如何匹配到一个 location 做处理呢？用一句话简单概括 Nginx 的 location 匹配规则是：“正则 location ”让步 “普通 location”的严格精确匹配结果；但覆盖 “普通 location ”的最大前缀匹配结果。理解这句话，我想通过下面的实例来说明。

* **先普通 location ，再正则 location**
周边不少童鞋告诉我， nginx 是“先匹配正则 location 再匹配普通 location ”，其实这是一个误区， nginx 其实是“先匹配普通 location ，再匹配正则 location ”，但是普通 location 的匹配结果又分两种：一种是“严格精确匹配”，官方英文说法是“ exact match ”；另一种是“最大前缀匹配”，官方英文说法是“ Literal strings match the beginning portion of the query – the most specific match will be used. ”。我们做个实验：

例题 1 ：假设 nginx 的配置如下
```
server {
       listen       9090;
       server_name  localhost;
       location / {
           root   html;
           index  index.html index.htm;
           deny all;
       }
       location ~ \.html$ {
           allow all;
       }
}
```
附录 nginx 的目录结构是： nginx->html->index.html
上述配置的意思是： location / {… deny all;} 普通 location 以“ / ”开始的 URI 请求（注意任何 HTTP 请求都必然以“/ ”开始，所以“ / ”的意思是所有的请求都能被匹配上），都拒绝访问； location ~\.html$ {allow all;} 正则 location以 .html 结尾的 URI 请求，都允许访问。

测试结果：
```bash
[root@web108 ~]# curl http://localhost:9090/
<html>
<head><title>403 Forbidden</title></head>
<body bgcolor=”white”>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.1.0</center>
</body>
</html>

[root@web108 ~]# curl http://localhost:9090/index.html
<html>
<head>
<title>Welcome to nginx!</title>
</head>
<body bgcolor=”white” text=”black”>
<center><h1>Welcome to nginx!</h1></center>
</body>
</html>

[root@web108 ~]# curl http://localhost:9090/index_notfound.html
<html>
<head><title>404 Not Found</title></head>
<body bgcolor=”white”>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.1.0</center>
</body>
</html>
```
测试结果如下：

URI请求 | HTTP响应
:--- | :---:
curl http://localhost:9090/ | 403 Forbidden
curl http://localhost:9090/index.html|Welcome to nginx!
curl http://localhost:9090/index_notfound.html|404 Not Found

`curl http://localhost:9090/` 的结果是“ 403 Forbidden ”，说明被匹配到“ location / {..deny all;} ”了，原因很简单HTTP 请求 GET / 被“严格精确”匹配到了普通 location / {} ，则会停止搜索正则 location ；

`curl http://localhost:9090/index.html` 结果是“ Welcome to nginx! ”，说明没有被“ location / {…deny all;} ”匹配，否则会 403 Forbidden ，但 /index.html 的确也是以“ / ”开头的，只不过此时的普通 location / 的匹配结果是“最大前缀”匹配，所以 Nginx 会继续搜索正则 location ， location ~ \.html$ 表达了以 .html 结尾的都 allow all; 于是接着就访问到了实际存在的 index.html 页面。

`curl http://localhost:9090/index_notfound.html`   同样的道理先匹配 location / {} ，但属于“普通 location 的最大前缀匹配”，于是后面被“正则 location ” location ~ \.html$ {} 覆盖了，最终 allow all ； 但的确目录下不存在index_notfound.html 页面，于是 404 Not Found 。

如果此时我们访问 http://localhost:9090/index.txt 会是什么结果呢？显然是 deny all ；因为先匹配上了 location / {..deny all;} 尽管属于“普通 location ”的最大前缀匹配结果，继续搜索正则 location ，但是 /index.txt 不是以 .html结尾的，正则 location 失败，最终采纳普通 location 的最大前缀匹配结果，于是 deny all 了。
```bash
[root@web108 ~]# curl http://localhost:9090/index.txt
<html>
<head><title>403 Forbidden</title></head>
<body bgcolor=”white”>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.1.0</center>
</body>
</html>
```
* **普通 location 的“隐式”严格匹配**

例题 2 ：我们在例题 1 的基础上增加精确配置

```
server {
       listen       9090;
       server_name  localhost;
       location /exact/match.html {
           allow all;
       }
       location / {
           root   html;
           index  index.html index.htm;
           deny all;
       }
       location ~ \.html$ {
           allow all;
       }
}
```
测试请求：
```bash
[root@web108 ~]# curl http://localhost:9090/exact/match.html
<html>
<head><title>404 Not Found</title></head>
<body bgcolor=”white”>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.1.0</center>
</body>
</html>
```

结果进一步验证了“普通 location ”的“严格精确”匹配会终止对正则 location 的搜索。这里我们小结下“普通 location”与“正则 location ”的匹配规则：先匹配普通 location ，再匹配正则 location ，但是如果普通 location 的匹配结果恰好是“严格精确（ exact match ）”的，则 nginx 不再尝试后面的正则 location ；如果普通 location 的匹配结果是“最大前缀”，则正则 location 的匹配覆盖普通 location 的匹配。也就是前面说的“正则 location 让步普通location 的严格精确匹配结果，但覆盖普通 location 的最大前缀匹配结果”。

* **普通 location 的“显式”严格匹配和“ ^~ ” 前缀**

上面我们演示的普通 location 都是不加任何前缀的，其实普通 location 也可以加前缀：“ ^~ ”和“ = ”。其中“ ^~”的意思是“非正则，不需要继续正则匹配”，也就是通常我们的普通 location ，还会继续搜索正则 location （恰好严格精确匹配除外），但是 nginx 很人性化允许配置人员告诉 nginx 某条普通 location ，无论最大前缀匹配，还是严格精确匹配都终止继续搜索正则 location ；而“ = ”则表达的是普通 location 不允许“最大前缀”匹配结果，必须严格等于，严格精确匹配。

例题 3 ：“ ^~ ”前缀的使用
```bash
server {
       listen       9090;
       server_name  localhost;
       location /exact/match.html {
           allow all;
       }
       location ^~ / {
           root   html;
           index  index.html index.htm;
           deny all;
       }
       location ~ \.html$ {
           allow all;
       }
}
```
把例题 2 中的 location / {} 修改成 location ^~ / {} ，再看看测试结果：

URL请求 | 修改前 | 修改后
---- | :----: | :----:
curl http://localhost:9090/ | 403 Forbidden | 403 Forbidden
curl http://localhost:9090/index.html | Welcome to nginx! | 403 Forbidden
curl http://localhost:9090/index_notfound.html | 404 Not Found | 403 Forbidden
curl http://localhost:9090/exact/match.html | 404 Not Found | 404 Not Found

除了 GET /exact/match.html 是 404 Not Found ，其余都是 403 Forbidden ，原因很简单所有请求都是以“ / ”开头，所以所有请求都能匹配上“ / ”普通 location ，但普通 location 的匹配原则是“最大前缀”，所以只有/exact/match.html 匹配到 location /exact/match.html {allow all;} ，其余都 location ^~ / {deny all;} 并终止正则搜索。

例题 4 ：“ = ”前缀的使用
```bash
server {
       listen       9090;
       server_name  localhost;
       location /exact/match.html {
           allow all;
       }
       location = / {
           root   html;
           index  index.html index.htm;
           deny all;
       }
       location ~ \.html$ {
           allow all;
       }
}
```
例题 4 相对例题 2 把 location / {} 修改成了 location = / {} ，再次测试结果：

URL请求 | 修改前 | 修改后
---- | :----: | :----:
curl http://localhost:9090/ | 403 Forbidden | 403 Forbidden
curl http://localhost:9090/index.html | Welcome to nginx! | Welcome to nginx!
curl http://localhost:9090/index_notfound.html | 404 Not Found | 404 Not Found
curl http://localhost:9090/exact/match.html | 404 Not Found | 404 Not Found
curl http://localhost:9090/test.jsp | 403 Forbidden | 404 Not Found

最能说明问题的测试是 GET /test.jsp ，实际上 /test.jsp 没有匹配正则 location （ location ~\.html$ ），也没有匹配 location = / {} ，如果按照 location / {} 的话，会“最大前缀”匹配到普通 location / {} ，结果是 deny all 。

* **正则 location 与编辑顺序**
location 的指令与编辑顺序无关，这句话不全对。对于普通 location 指令，匹配规则是：最大前缀匹配（与顺序无关），如果恰好是严格精确匹配结果或者加有前缀“ ^~ ”或“ = ”（符号“ = ”只能严格匹配，不能前缀匹配），则停止搜索正则 location ；但对于正则 location 的匹配规则是：按编辑顺序逐个匹配（与顺序有关），只要匹配上，就立即停止后面的搜索。

```bash
配置 3.1

server {
       listen       9090;
       server_name  localhost;
       location ~ \.html$ {
           allow all; 
       }  
       location ~ ^/prefix/.*\.html$ {
           deny all;  
       }  
}
配置 3.2

server {
       listen       9090;
       server_name  localhost;     
       location ~ ^/prefix/.*\.html$ {
           deny all;  
       }            
       location ~ \.html$ {
           allow all; 
       } 
}
```
测试结果：

URL请求 | 配置3.1 | 配置3.2
---- | :----: | :----:
curl http://localhost:9090/regextest.html | 404 Not Found | 404 Not Found
curl http://localhost:9090/prefix/regextest.html | 404 Not Found | 403 Forbidden

解释：
`Location ~ ^/prefix/.*\.html$ {deny all;}` 表示正则 location 对于以 /prefix/ 开头， .html 结尾的所有 URI 请求，都拒绝访问；   `location ~\.html${allow all;}` 表示正则 location 对于以 .html 结尾的 URI 请求，都允许访问。 实际上，prefix 的是 ~\.html$ 的子集。

在“配置 3.1 ”下，两个请求都匹配上 `location ~\.html$ {allow all;}` ，并且停止后面的搜索，于是都允许访问， 404 Not Found ；在“配置 3.2 ”下， /regextest.html 无法匹配 prefix ，于是继续搜索 ~\.html$ ，允许访问，于是 404 Not Found ；然而 /prefix/regextest.html 匹配到 prefix ，于是 deny all ， 403 Forbidden 。
```
配置 3.3

server {
       listen       9090;
       server_name  localhost; 
       location  /prefix/ {
               deny all;  
       }           
       location  /prefix/mid/ {
               allow all; 
       }  
}
配置 3.4

server {
       listen       9090;
       server_name  localhost;    
       location  /prefix/mid/ {
               allow all; 
       }  
        location  /prefix/ {
               deny all;  
       }  
}
```
测试结果：

URL请求 | 配置3.3 | 配置3.4
---- | :----: | :----:
curl http://localhost:9090/prefix/t.html | 403 Forbidden | 403 Forbidden
curl http://localhost:9090/prefix/mid/t.html | 404 Not Found | 404 Not Found

测试结果表明：普通 location 的匹配规则是“最大前缀”匹配，而且与编辑顺序无关。

* **“@” 前缀 Named Location 使用**
REFER:  <http://wiki.nginx.org/HttpCoreModule#error_page>
假设配置如下：
```bash
server {
       listen       9090;
       server_name  localhost;
       location  / {
           root   html;
           index  index.html index.htm;
           allow all;
       }
       #error_page 404 http://www.baidu.com # 直接这样是不允许的
       error_page 404 = @fallback;
	   
       location @fallback {
           proxy_pass http://www.baidu.com;
       }
}
```
上述配置文件的意思是：如果请求的 URI 存在，则本 nginx 返回对应的页面；如果不存在，则把请求代理到baidu.com 上去做个弥补（注： nginx 当发现 URI 对应的页面不存在， HTTP_StatusCode 会是 404 ，此时error_page 404 指令能捕获它）。

测试一：

```bash
[root@web108 ~]# curl http://localhost:9090/nofound.html -i
HTTP/1.1 302 Found
Server: nginx/1.1.0
Date: Sat, 06 Aug 2011 08:17:21 GMT
Content-Type: text/html; charset=iso-8859-1
Location: http://localhost:9090/search/error.html
Connection: keep-alive
Cache-Control: max-age=86400
Expires: Sun, 07 Aug 2011 08:17:21 GMT
Content-Length: 222

<!DOCTYPE HTML PUBLIC “-//IETF//DTD HTML 2.0//EN”>
<html><head>
<title>302 Found</title>
</head><body>
<h1>Found</h1>
<p>The document has moved <a href=”http://www.baidu.com/search/error.html”>here</a>.</p>
</body></html>
```
当我们 GET /nofound.html 发送给本 nginx ， nginx 找不到对应的页面，于是 error_page 404 = @fallback ，请求被代理到 http://www.baidu.com ，于是 nginx 给 http://www.baidu.com 发送了 GET /nofound.html ，但/nofound.html 页面在百度也不存在，百度 302 跳转到错误页。
直接访问 http://www.baidu.com/nofound.html 结果：

```bash
[root@web108 ~]# curl http://www.baidu.com/nofound.html -i
HTTP/1.1 302 Found
Date: Sat, 06 Aug 2011 08:20:05 GMT
Server: Apache
Location: http://www.baidu.com/search/error.html
Cache-Control: max-age=86400
Expires: Sun, 07 Aug 2011 08:20:05 GMT
Content-Length: 222
Connection: Keep-Alive
Content-Type: text/html; charset=iso-8859-1
 
<!DOCTYPE HTML PUBLIC “-//IETF//DTD HTML 2.0//EN”>
<html><head>
<title>302 Found</title>
</head><body>
<h1>Found</h1>
<p>The document has moved <a href=”http://www.baidu.com/search/error.html”>here</a>.</p>
</body></html>
```
测试二：访问一个 nginx 不存在，但 baidu 存在的页面
```bash
[root@web108 ~]# curl http://www.baidu.com/duty/ -i
HTTP/1.1 200 OK
Date: Sat, 06 Aug 2011 08:21:56 GMT
Server: Apache
P3P: CP=” OTI DSP COR IVA OUR IND COM ”
P3P: CP=” OTI DSP COR IVA OUR IND COM ”
Set-Cookie: BAIDUID=5C5D2B2FD083737A0C88CA7075A6601A:FG=1; expires=Sun, 05-Aug-12 08:21:56 GMT; max-age=31536000; path=/; domain=.baidu.com; version=1
Set-Cookie: BAIDUID=5C5D2B2FD083737A2337F78F909CCB90:FG=1; expires=Sun, 05-Aug-12 08:21:56 GMT; max-age=31536000; path=/; domain=.baidu.com; version=1
Last-Modified: Wed, 05 Jan 2011 06:44:53 GMT
ETag: “d66-49913b8efe340″
Accept-Ranges: bytes
Content-Length: 3430
Cache-Control: max-age=86400
Expires: Sun, 07 Aug 2011 08:21:56 GMT
Vary: Accept-Encoding,User-Agent
Connection: Keep-Alive
Content-Type: text/html

<!DOCTYPE HTML PUBLIC “-//W3C//DTD HTML 4.01 Transitional//EN”
“http://www.w3.org/TR/html4/loose.dtd”>
。。。。
</body>
</html>
```
显示，的确百度这个页面是存在的。

```bash
[root@web108 ~]# curl http://localhost:9090/duty/ -i
HTTP/1.1 200 OK
Server: nginx/1.1.0
Date: Sat, 06 Aug 2011 08:23:23 GMT
Content-Type: text/html
Connection: keep-alive
P3P: CP=” OTI DSP COR IVA OUR IND COM ”
P3P: CP=” OTI DSP COR IVA OUR IND COM ”
Set-Cookie: BAIDUID=8FEF0A3A2C31D277DCB4CC5F80B7F457:FG=1; expires=Sun, 05-Aug-12 08:23:23 GMT; max-age=31536000; path=/; domain=.baidu.com; version=1
Set-Cookie: BAIDUID=8FEF0A3A2C31D277B1F87691AFFD7440:FG=1; expires=Sun, 05-Aug-12 08:23:23 GMT; max-age=31536000; path=/; domain=.baidu.com; version=1
Last-Modified: Wed, 05 Jan 2011 06:44:53 GMT
ETag: “d66-49913b8efe340″
Accept-Ranges: bytes
Content-Length: 3430
Cache-Control: max-age=86400
Expires: Sun, 07 Aug 2011 08:23:23 GMT
Vary: Accept-Encoding,User-Agent

<!DOCTYPE HTML PUBLIC “-//W3C//DTD HTML 4.01 Transitional//EN”
“http://www.w3.org/TR/html4/loose.dtd”>
<html>
。。。
</body>
</html>
```
当 `curl http://localhost:9090/duty/ -i` 时， nginx 没找到对应的页面，于是 error_page = @fallback ，把请求代理到 baidu.com 。注意这里的 error_page = @fallback 不是靠重定向实现的，而是所说的“ internally redirected （forward ）”。

### proxy_pass URL 末尾加与不加/（斜线）的区别
>Proxy_pass末尾带”/”和不带是有区别的：
不带斜杠转发的是除hostname以外的部分，包括目录。可以使用正则表达式匹配location，且任意正则匹配成功后，转发的都是完整目录路径。
带斜杠转发的是除hostname及目录外的所有部分。不能使用正则表达式匹配location块，只能使用完整路径名准确匹配。

```
配置1
location /tobaidu {
    proxy_pass http://127.0.0.1:8087;
}
```
测试结果：

请求URL | 请求结果
---- | ----
curl http://127.0.0.1/tobaidu | http://127.0.0.1:8087/tobaidu
curl http://127.0.0.1/tobaidu/ | http://127.0.0.1:8087/tobaidu/
curl http://127.0.0.1/tobaidu/xxxx | http://127.0.0.1:8087/tobaidu/xxxx

```
配置2
location /tobaidu {
    proxy_pass http://127.0.0.1:8087/define;
}
```
测试结果：

请求URL | 请求结果
---- | ----
curl http://127.0.0.1/tobaidu | http://127.0.0.1:8087/define
curl http://127.0.0.1/tobaidu/ | http://127.0.0.1:8087/define/
curl http://127.0.0.1/tobaidu/xxxx | http://127.0.0.1:8087/define/xxxx

```
配置3
location /tobaidu/ {
    proxy_pass http://127.0.0.1:8087;
}
```
测试结果：

请求URL | 请求结果
---- | ----
curl http://127.0.0.1/tobaidu | 重定向到http://127.0.0.1/tobaidu/
curl http://127.0.0.1/tobaidu/ | http://127.0.0.1:8087/tobaidu/
curl http://127.0.0.1/tobaidu/xxxx | http://127.0.0.1:8087/tobaidu/xxxx

```
配置4
location /tobaidu/ {
    proxy_pass http://127.0.0.1:8087/define;
}
```
测试结果：

请求URL | 请求结果
---- | ----
curl http://127.0.0.1/tobaidu | 重定向到http://127.0.0.1/tobaidu/
curl http://127.0.0.1/tobaidu/ | http://127.0.0.1:8087/define
curl http://127.0.0.1/tobaidu/xxxx | http://127.0.0.1:8087/define/xxxx

```
配置5
location /tobaidu {
    proxy_pass http://127.0.0.1:8087/;
}
```
测试结果：

请求URL | 请求结果
---- | ----
curl http://127.0.0.1/tobaidu | http://127.0.0.1:8087/
curl http://127.0.0.1/tobaidu/ | http://127.0.0.1:8087//
curl http://127.0.0.1/tobaidu/xxxx | http://127.0.0.1:8087//xxxx

```
配置6
location /tobaidu {
    proxy_pass http://127.0.0.1:8087/define/;
}
```
测试结果：

请求URL | 请求结果
---- | ----
curl http://127.0.0.1/tobaidu | http://127.0.0.1:8087/define/
curl http://127.0.0.1/tobaidu/ | http://127.0.0.1:8087/define//
curl http://127.0.0.1/tobaidu/xxxx | http://127.0.0.1:8087/define//xxxx

```
配置7
location /tobaidu/ {
    proxy_pass http://127.0.0.1:8087/;
}
```
测试结果：

请求URL | 请求结果
---- | ----
curl http://127.0.0.1/tobaidu | 重定向到http://127.0.0.1/tobaidu/
curl http://127.0.0.1/tobaidu/ | http://127.0.0.1:8087/
curl http://127.0.0.1/tobaidu/xxxx | http://127.0.0.1:8087/xxxx

```
配置8
location /tobaidu/ {
    proxy_pass http://127.0.0.1:8087/define/;
}
```
测试结果：

请求URL | 请求结果
---- | ----
curl http://127.0.0.1/tobaidu | 重定向到http://127.0.0.1/tobaidu/
curl http://127.0.0.1/tobaidu/ | http://127.0.0.1:8087/define/
curl http://127.0.0.1/tobaidu/xxxx | http://127.0.0.1:8087/define/xxxx

**结论**
*URL符合 `protocol://ip:port` 同时结尾不加/,则nginx会代理匹配路径部分,否则不代理匹配路径,同时自动添加不匹配路径”部分”,比如/tobaidu/xxxx的/xxxx部分*