layout: post
title: Markdown格式书写
author: Pyker
categories: markdown
img: /images/bar/markdown.jpg
tags:
  - markdown
top: false
cover: false
date: 2017-07-23 10:20:00
---
## 区块元素

### 标题

Markdown 支持两种标题的语法，类似 Setext 和类 atx 形式。
类 Setext 形式是用底线的形式，利用 = （最高阶标题）和 - （第二阶标题），例如：
```
This is an H1
=============
This is an H2
-------------
```
也可以是类 Atx 形式则是在行首插入 1 到 6 个 # ，对应到标题 1 到 6 阶，例如：
```
# 这是 H1
## 这是 H2
###### 这是 H6
```
### 区块引用 Blockquotes

区块引用是使用类似 email 中用 > 的引用方式。例如:
```
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
>
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
```
区块引用可以嵌套（例如：引用内的引用），只要根据层次加上不同数量的 > ：
```
> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.
```
引用的区块内也可以使用其他的 Markdown 语法，包括标题、列表、代码区块等：
```
> ### 这是一个标题。
> 
> 1.   这是第一行列表项。
> 2.   这是第二行列表项。
> 
> 给出一些例子代码：
> 
>     return shell_exec("echo $input | $markdown_script");
```
### 列表

Markdown 支持有序列表和无序列表。
* 无序列表使用星号、加号或是减号作为列表标记：

```
*   Red
*   Green
*   Blue

等同于：
+   Red
+   Green
+   Blue

也等同于：
-   Red
-   Green
-   Blue
```

* 有序列表则使用数字接着一个英文句点：

```
1.  Bird
2.  McHale
3.  Parish
```
>要让列表看起来更漂亮，你可以把内容用固定的缩进整理好：
>
>	 * Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
>	   Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
>	   viverra nec, fringilla in, laoreet vitae, risus.
>	 * Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
>	   Suspendisse id sem consectetuer libero luctus adipiscing.

>列表项目可以`包含多个段落`， 每个项目下的段落都必须缩进 4 个空格或是 1 个制表符：
>
>	 1.  This is a list item with two paragraphs. Lorem ipsum dolor
>         sit amet, consectetuer adipiscing elit. Aliquam hendrerit
>         mi posuere lectus.
>
>         Vestibulum enim wisi, viverra nec, fringilla in, laoreet
>         vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
>         sit amet velit.
>
>	 2. Suspendisse id sem consectetuer libero luctus adipiscing.

>如果要在列表项目内放进引用，那 > 就需要缩进：
>
>	 * A list item with a blockquote:
>
>	   > This is a blockquote
>	   > inside a list item.

>如果要放代码区块的话，该`区块就需要缩进两次`，也就是 8 个空格或是 2 个制表符：
>
> 	*   一列表项包含一个列表区块：
>
>         <代码写在这>

### 代码区块

和程序相关的写作或是标签语言原始码通常会有已经排版好的代码区块，通常这些区块我们并不希望它以一般段落文件的方式去排版，而是照原来的样子显示，要在 Markdown 中建立代码区块很简单，只要代码块每行简单地缩进 4 个空格或是 1 个制表符就可以，例如，下面的输入：
```
这是一个普通段落：

    这是一个代码区块。
```
>markdown也支持html的源码格式，只需要复制贴上，再加上缩进就可以了，剩下的 Markdown 都会帮你处理，例如：
>
>  	 <div class="footer">
> 	  	 &copy; 2004 Foo Corporation
>  	 </div>

代码区块中，一般的 Markdown 语法不会被转换，像是星号便只是星号，这表示你可以很容易地以 Markdown 语法撰写 Markdown 语法相关的文件。

### 分割线

你可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。你也可以在星号或是减号中间插入空格。下面每种写法都可以建立分隔线：
```
* * *

***

*****

- - -
```
### 表格
在markdown中也支持表格格式，格式如下：
```
菜单1 | 菜单2 | 菜单3 | 菜单n
----: | :---：| :---- | :----
内容1 | 内容1 | 内容1 | 内容1
内容2 | 内容2 | 内容2 | 内容2
内容3 | 内容3 | 内容3 | 内容3
```
Markdown 插入的表格，单元格中默认左对齐；表头单元格中的内容会一直居中对齐，可以使用：来设置对齐方式，`:---:` 居中对齐，`---:` 右对齐， `:---` 左对齐，`-` 的个数不限制。

## 区段元素

### 链接
Markdown 支持两种形式的链接语法： `行内式` 和`参考式` 两种形式。
不管是哪一种，链接文字都是用 [方括号] 来标记。
要建立一个`行内式` 的链接，只要在方块括号后面紧接着圆括号并插入网址链接即可，如果你还想要加上链接的 title 文字，只要在网址后面，用双引号把 title 文字包起来即可，例如：
```
This is [an example](http://example.com/ "Title") inline link.
[This link](http://example.net/) has no title attribute.
```
> 如果你是要链接到同样主机的资源，你可以使用相对路径：
>
>	 See my [About](/about/) page for details.

`参考式` 的链接是在链接文字的括号后面再接上另一个方括号，而在第二个方括号里面要填入用以辨识链接的标记,接着，在文件的任意处，你可以把这个标记的链接内容定义出来：：
```
This is [an example][id] reference-style link.
[id]: http://example.com/  "Optional Title Here"
```

链接内容定义的形式为：

* 方括号（前面可以选择性地加上至多三个空格来缩进），里面输入链接文字
* 接着一个冒号
* 接着一个以上的空格或制表符
* 接着链接的网址
* 选择性地接着 title 内容，可以用单引号、双引号或是括弧包着

下面这三种链接的定义都是相同：
```
[foo]: http://example.com/  "Optional Title Here"
[foo]: http://example.com/  'Optional Title Here'
[foo]: http://example.com/  (Optional Title Here)
```
**请注意：** 有一个已知的问题是 Markdown.pl 1.0.1 会忽略单引号包起来的链接 title。

### 强调
Markdown 使用星号`（*）和底线（_）作为标记强调字词` 的符号，被 * 或 _ 包围的字词会被转成用 <em> 标签包围，用`两个 * 或 _ ` 包起来的话，则会被转成 <strong>，被`三个星号（*）` 包起来则为倾斜和加粗文字，被两个`波浪线（~~）` 包为起来是要在文字上加删除线。例如：
```
*single asterisks*

_single underscores_

**double asterisks**

__double underscores__

***double underscores***

~~double underscores~~
```
你可以随便用你喜欢的样式，唯一的限制是，你用什么符号开启标签，就要用什么符号结束。
<font color=#FF0000 >但是如果你的 * 和 _ 两边都有空白的话，它们就只会被当成普通的符号。</font>
>如果要在文字前后直接插入普通的星号或底线，你可以用反斜线：
>
>	 \*this text is surrounded by literal asterisks\*

### 代码
* 单行短句

如果要标记一小段行内代码，你可以用反引号把它包起来\`  \`，例如：
```
Use the `printf()` function.
```
> 如果要在代码区段内插入反引号，你可以用多个反引号来开启和结束代码区段：
>
>	 ``There is a literal backtick (`) here.``

* 多行代码块 

	  ```
	  代码块1
	  代码块2
	  代码块3
	  ```
其中\`\`\`也可以用~~~代替，且\`\`\`后也支持语言关键字书写，使代码块格式化。例如\`\`\`bash\`\`\`

*语言对应关键字请参照本文最末尾。*
	  
### 图片
很明显地，要在纯文字应用中设计一个「自然」的语法来插入图片是有一定难度的。Markdown 使用一种和链接很相似的语法来标记图片，同样也允许两种样式： `行内式` 和`参考式` 。
行内式的图片语法看起来像是：
```
![Alt text](/path/to/img.jpg)
![Alt text](/path/to/img.jpg "Optional title")
```
详细叙述如下：

* 一个惊叹号 !
* 接着一个方括号，里面放上图片下面的说明文字，相当于对图片内容的解释。
* 接着一个普通括号，里面放上图片的网址，最后还可以用引号包住并加上 选择性的 'title' 文字。

`参考式` 的图片语法则长得像这样：
```
![Alt text][id]
[id]: url/to/image  "Optional title attribute"
```
「id」是图片参考的名称，图片参考的定义方式则和连结参考一样：

### 字体和颜色
Markdown是一种可以使用普通文本编辑器编写的标记语言，通过类似HTML的标记语法，它可以使普通文本内容具有一定的格式。但是它`本身是不支持修改字体、字号与颜色` 等功能的！
为了使其修改字体和颜色，我们可以用html语法代替，如：
```html
<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=#FF0000 size=3 face="黑体">这是一行红色3号大小的黑体文本</font>
<font color=#00ffff size=72>这是16位颜色值表示</font>
<font color=gray size=72>也可以用颜色的英文单词表示</font>
```
>Size：规定文本的尺寸大小。可能的值：从 1 到 7 的数字。浏览器默认值是 3。
颜色16进制值可以参考：<https://www.114la.com/other/rgb.htm>

### note和label颜色
下面是note显示的颜色：
<div class="note default"><p>default</p></div>
<div class="note primary"><p>primary</p></div>
<div class="note success"><p>success</p></div>
<div class="note info"><p>info</p></div>
<div class="note warning"><p>warning</p></div>
<div class="note danger"><p>danger</p></div>
<div class="note danger no-icon"><p>danger no-icon</p></div>
```html
<div class="note default"><p>default</p></div>
<div class="note primary"><p>primary</p></div>
<div class="note success"><p>success</p></div>
<div class="note info"><p>info</p></div>
<div class="note warning"><p>warning</p></div>
<div class="note danger"><p>danger</p></div>
<div class="note danger no-icon"><p>danger no-icon</p></div>
```
下面是label显示的颜色
{% label primary@primary %}
{% label default@default %}
{% label success@success %}
{% label info@info %}
{% label warning@warning %}
{% label danger@danger %}
```html
{% label primary@primary %}
{% label default@default %}
{% label success@success %}
{% label info@info %}
{% label warning@warning %}
{% label danger@danger %}
```
## 其他

### 自动链接
Markdown 支持以比较简短的自动链接形式来处理网址和电子邮件信箱，只要是用方括号包起来， Markdown 就会自动把它转成链接。一般网址的链接文字就和链接地址一样，例如：
```
<http://example.com/>
```
Markdown 会转为：

	 <a href="http://example.com/">http://example.com/</a>
### 反斜杠
Markdown 可以利用反斜杠来插入一些在语法中有其它意义的符号，例如：如果你想要用星号加在文字旁边的方式来做出强调效果（但不用 <em> 标签），你可以在星号的前面加上反斜杠：
```
\*literal asterisks\*
```
Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：
```
\   反斜线
`   反引号
*   星号
_   底线
{}  花括号
[]  方括号
()  括弧
#   井字号
+   加号
-   减号
.   英文句点
!   惊叹号
```

***
语言关键字

语言名 | 关键字
:----: | :----:
Bash | bash
CoffeeScript | coffeescript
C++ | cpp
C# | cs
CSS | css
Diff | diff
HTTP | http
Ini | ini
Java | java
JavaScript | javascript
JSON | json
XML | xml
Makefile | makefile
Markdown | markdown
Objective-C | objectivec
Perl | perl
Python | python
Ruby | ruby
SQL | sql