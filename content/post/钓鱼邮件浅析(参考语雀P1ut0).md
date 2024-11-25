+++
title = '钓鱼邮件浅析(参考语雀P1ut0)'

description = "鱼叉是漏洞之外最常见的入口点"

tags = [ "技术" ]

date = 2024-11-20T18:42:55+08:00

+++

#### 一种未曾见过的钓鱼手法：使用iframe元素填充页面

![image.png](https://cdn.nlark.com/yuque/0/2021/png/302292/1609744440755-e8b1f0a9-633a-4b00-8c37-81eed0157565.png?x-oss-process=image%2Fformat%2Cwebp)

虽然但是，直接看url也能看出不是官方页面。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/302292/1609744855408-9f0dcac8-46f7-4355-aec2-64cadfd52c88.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_903%2Climit_0)

该页面实际上被一个满屏的iframe给填充了：

![image.png](https://cdn.nlark.com/yuque/0/2021/png/302292/1609744913369-35598eed-f81f-46ca-91ae-b3893d302c41.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_959%2Climit_0)

只是在直接的钓鱼页面过程中加上了一层跳转，实际上该是黑的域名还是黑的。

#### 直接投递exe类钓鱼邮件

一般都会对样本进行加密压缩规避杀软，同时将解压密码写在正文中。APT-Q-15这样做过，虽然投递的是msi。

#### 钓鱼邮件下发漏洞文档

最常见的CVE2017-11822，从C2远程加载exe

#### Macro4.0宏文档投递

VBA（Visual Basic for Applications）是为Microsoft Office软件开发的一种宏语言。它可以用于各种Office应用程序，如Excel、Word、PowerPoint等。VBA有强大的内置函数和对象模型，可以轻松地扩展Office应用程序的功能，从而提高工作效率。VBA也可以用于创建自己的用户界面和编写各种业务应用程序。

VBS（Visual Basic Script）是一种通用的脚本语言，它通常用于Windows操作系统中的各种任务和自动化操作。VBS通常用于运行脚本文件、自动化常规任务、设置环境变量、执行Windows命令等等。VBS相比于VBA，不依赖于Office应用程序，并且可以执行操作系统级别的任务。

但VBS不等于XLM宏。

| VBA                  | XLM宏             |
| -------------------- | ----------------- |
| Excel5.0技术         | Excel4.0技术      |
| 依赖于office组件执行 | 依赖于wscript.exe |
| 语法相同             | 语法相同          |

XLM宏的创建比较简单，直接在某个工作表右键选择插入MS Excel4.0宏表。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/302292/1608619383600-cee144a8-fb32-4a26-bd4c-1f00ff1ce473.png?x-oss-process=image%2Fformat%2Cwebp)

Excel则会自动创建一个名为Macro1的新工作表，中文系统下为<宏1>，在创建的新工作表中，可以执行宏指令。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/302292/1608620265057-872d8251-4528-4489-81d1-3647db676d02.png?x-oss-process=image%2Fformat%2Cwebp)

使用XLM宏比常见的VBA宏具有更好的免杀性。原因主要是：

XLM宏和VBA宏的设计理念不同，导致了宏代码在文件结构中的呈现不同。和VBA宏一样的是，在文件打开时，Excel依旧会提醒用户

> Because of your security settings, macros have been disabled. To run macros, you need to reopen this workbook, and then choose to enable macros. For more information about enabling macros, click Help.

让用户决定是否要启用宏。**不同的是，当用户单击启用之后，ALT + F11打开宏代码窗口，却并不能看到宏代码。也不能通过一些常见的宏代码提取工具检测分析宏。这是因为默认情况下，XLM宏代码存储在xl\Macrosheets\文件夹下的Sheet1.xml中。打开该xml文件，可以清晰的看到刚才在Excel工作表中插入的宏代码。**

![img](https://cdn.nlark.com/yuque/0/2020/png/302292/1608621107692-98e018b7-8920-443b-a7c2-71f770e80331.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_750%2Climit_0)

