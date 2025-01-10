+++
title = '整理一些基础Web攻防知识'

description = "Web安全深度剖析"

tags = [ "技术" ]

date = 2022-01-08T15:19:02+08:00

+++

### SQL注入漏洞

SQL注入的原理：用户输入的数据被解释器执行

SQL注入的分类

1. 数字型注入：多出现在ASP/PHP等弱语言中，弱类型语言会自动推导变量类型。例如id=8，PHP自动推导变量id的数据类型为int类型。id=8 and 1=1，则会推导为string类型，这是弱语言类型的特性。强类型语言的数字型注入漏洞会比弱类型语言少很多。
2. 字符型注入：输入参数为字符串时，称为字符型注入。最大区别在于：数字型注入不需要单引号闭合，字符串类型一般要使用单引号来闭合。字符型注入最关键的是如何闭合SQL语句以及注释多余的代码。当查询内容为字符串时，SQL代码如下`select * from table where username = 'admin'`，当攻击者进行SQL注入时，直接输入`select * from table where username = 'admin and 1=1' `会被当成查询语句，此处想要注入必须注意字符串闭合问题：`select * from table where username = 'admin' and 1=1 --`
3. SQL注入的分类：POST注入，即注入字段在POST数据中。Cookie注入，注入字段在Cookie数据中。延时注入，使用数据库延时特性注入。搜索注入，注入处为搜索的地点。base64注入，注入字符串需要经过base64加密。

### 上传漏洞

### XSS跨站脚本漏洞

XXS又叫CSS即Cross Site Scripting，即跨站脚本攻击。指的是攻击者在网页中嵌入客户端脚本，通常是JS编写的恶意代码。当用户使用浏览器浏览该网页时，恶意代码将会被用户的浏览器执行。

JS加载外部的代码文件可以是任意扩展名（无扩展名也可以），如`<script src="http://www.secbug.org/x.jpg"></script>`，即使文件为图片扩展名x.jpg，只要其中包含JS代码就会被执行。

### 命令执行漏洞

### 文件包含漏洞

### 其它漏洞
