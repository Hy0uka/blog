+++
title = 'Living Off the Land Binaries'

description = "包括白利用在内"

tags = [ "技术" ]

date = 2024-10-28T18:07:46+08:00

+++

#### 什么样的程序才能称之为LOLBins？

- 可以是带有Microsoft签名的二进制文件，也可以是Microsoft系统目录中的二进制文件。
- 可以是第三方认证签名的程序。
- 具有对APT或红队有用的功能。
- 该程序除正常的功能之外，还可以做意料之外的行为（如恶意代码执行，UAC绕过）

#### LOLBins常见的利用

**Cmd.exe**

**Powershell.exe**

**Rundll32.exe**

**Wmic.exe**

WMIC扩展WMI（Windows Management Instrumentation，Windows管理工具），执行“wmic”命令启动WMIC命令行环境。这个命令可以在XP或 .NET Server的标准命令行解释器（cmd.exe）、Telnet会话或“运行”对话框中执行。这些启动方法可以在本地使用，也可以通过.NET Server终端服务会话使用。

**Mshta.exe**

文全称Microsoft HTML Application，可翻译为微软超文本标记语言应用。

**Cscript.exe**

Windows Script Host引擎，用于加载wsf，.vbs，.js等脚本。

**Regsvr32.exe**

用于注册COM组件，是 Windows 系统提供的用来向系统注册控件或者卸载控件的命令。

**Cerutil.exe**

可用于在Windows中管理证书。使用此程序可以在Windows中安装，备份，删除，管理和执行与证书和证书存储相关的各种功能。程序有一特性是能够从远程URL下载证书或任何其他文件。

**Msiexec.exe**

系统进程，是Windows Installer的一部分。用于安装Windows Installer安装包（MSI）。

**Installutil.exe**

安装程序工具是一个命令行实用工具，你可以通过此工具执行指定程序集中的安装程序组件，从而安装和卸载服务器资源。程序将会寻找卸载程序中的InstallUtil()函数，进行调用。

**MSBuild.exe**

生成项目或解决方案文件。程序运行将会寻找该项目文件中指定的TaskName名称中对应的程序的Execute()函数，进行调用。

**Write.exe**

 系统写字板程序，程序加载需调用PROPSYS.dll中的PSCreateMemoryPropertyStore()函数,同时也加载config配置文件。

**Rekeywiz.exe**

加密文件系统证书管理向导程序，程序加载需调用mpr.dll。

#### 第三方认证签名文件利用

**WeChat.exe**

微信主程序，需调用WeChatWin.dll运行。
