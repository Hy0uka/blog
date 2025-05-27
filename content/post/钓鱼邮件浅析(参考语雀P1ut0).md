+++
title = '钓鱼邮件浅析(参考语雀P1ut0)'

description = "鱼叉是漏洞之外最常见的入口点"

tags = [ "APT" ]

date = 2024-11-20T18:42:55+08:00

+++

#### 一种未曾见过的钓鱼手法：使用iframe元素填充页面

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/11/140b75470ebc2561d9f4d3e671fa1ede.png)

虽然但是，直接看url也能看出不是官方页面。

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/11/647efb2bdfeb4192b10c8f7bde679b1c.png)

该页面实际上被一个满屏的iframe给填充了：

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/11/57d31d25d2f9dc189296bf2a0d047260.png)

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

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/11/f68d3f153c5dd09e71ecc92bea7f75b5.png)

Excel则会自动创建一个名为Macro1的新工作表，中文系统下为<宏1>，在创建的新工作表中，可以执行宏指令。

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/11/43351dcae816838c5f1cc6b52bdcfaa9.png)

使用XLM宏比常见的VBA宏具有更好的免杀性。原因主要是：

XLM宏和VBA宏的设计理念不同，导致了宏代码在文件结构中的呈现不同。和VBA宏一样的是，在文件打开时，Excel依旧会提醒用户

> Because of your security settings, macros have been disabled. To run macros, you need to reopen this workbook, and then choose to enable macros. For more information about enabling macros, click Help.

让用户决定是否要启用宏。**不同的是，当用户单击启用之后，ALT + F11打开宏代码窗口，却并不能看到宏代码。也不能通过一些常见的宏代码提取工具检测分析宏。这是因为默认情况下，XLM宏代码存储在xl\Macrosheets\文件夹下的Sheet1.xml中。打开该xml文件，可以清晰的看到刚才在Excel工作表中插入的宏代码。**

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2024/11/7d105156c0d16f5314ee44aee1a1c8e3.png)

### 钓鱼邮件伪造防范

#### 查看邮件头信息（Email Header）

邮件头包含了邮件的详细元数据，是真实发件人身份验证的关键。你可以查看以下几个字段：

#### 1. **Return-Path**

- 表示邮件退回的地址，攻击者通常会伪造这个地址。
- 与“From”字段不同则可疑。

#### 2. **Received**（多级）

- 显示邮件在传输过程中的每一跳（由哪个邮件服务器接收，再转发）。
- **从底部往上看**，看是否有可疑或不一致的来源 IP。

#### 3. **Message-ID**

- 合法邮件服务（如 Gmail）会生成以其域名结尾的 Message-ID，如 `<randomid@mail.gmail.com>`。
- 可疑的伪造邮件可能使用非标准或不一致的 Message-ID。

#### 检查域名验证机制（反伪造技术）

现代邮件系统用以下机制防止伪造：

#### 1. **SPF（Sender Policy Framework）**

- 用来验证某个 IP 是否有权限代表该域名发送邮件。
- 查看邮件头中的 `Received-SPF` 字段，值应为 `pass`。
  - 示例：`Received-SPF: pass (domain of example.com designates 192.0.2.1 as permitted sender)`

#### 2. **DKIM（DomainKeys Identified Mail）**

- 使用加密签名验证发件人域名。
- 邮件头中应有：`DKIM-Signature` 和验证结果如：`Authentication-Results: dkim=pass`

#### 3. **DMARC（Domain-based Message Authentication, Reporting & Conformance）**

- 基于 SPF 和 DKIM，给出统一策略判断邮件是否合法。
- 邮件头中应有类似：`Authentication-Results: dmarc=pass`
