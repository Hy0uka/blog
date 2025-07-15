+++
title = '样本分析经验Summary'

description = "Meta"

tags = [ "Meta" ]

date = 2025-07-07T17:49:48+08:00
draft = true

+++

1. Shellcode一般不会是最终的Backdoor，一般是用来过免杀的中间程序。最后的Backdoor一般是完整的DLL，可能是从内存解密后完成重定位进行加载，也可能是从C2下载后执行。

2. C#的程序作为Backdoor比较多。

3. Dnspy分析C#时，右键直接跳转到entry point。

4. C#加载DLL一般用反射型加载，在invoke处下断点。

5. CS的后门判断特征：1.CS有一个导出函数"ReflectiveLoader"。2.一般调用Winnet.dll中的InternetConnect函数进行回连。3.如果使用的是shellcode的话，在CS的4.3版本之前shellcode,C2是直接跟在shellcode末尾的。提取shellcode或stager中配置文件的工具：https://github.com/Sentinel-One/CobaltStrikeParser。一般红队会做免杀，基本都是加载器，可以等加载器把内容释放出来后，直接dump上传给引擎，让引擎判断。

6. exe文件拖入Dnspy后在侧边栏中名称为黄色，而dll文件拖入后在侧边栏中名称为白色。

7. 一般来说程序具有窗口(无论隐藏与否)时，main函数都是用来调用Application.Run()进行窗口的启动。一般来说，恶意操作都会在隐藏窗口类中进行，我们只需向下寻找即可。

8. C#写Loader，注意namespace要和dll原本的名称一致，以及传入参数。

9. 另外分享一个快速调试方式，经过前辈们分析底层函数发现一个关键函数：InvokeMethodFast()，此函数在运行dll之后会被调用，步入断点所在的代码，就可以直接进入dll进行调试。ps:这个方法可以一直运行下去，省了dump的麻烦，可谓非常厉害了。

10. 随便加载一个C#的exe（一般是安装器），右键选 与程序集合并。 然后选择待调试的dll，这时候dll的所有类、资源、引用都与exe合并到一起了。然后修改原exe的入口函数（右键编辑类）直接调用dll的入口函数。最后Ctrl+Shift+S保存，再调试exe就是直接跑dll的代码了。/我自己测试了一个发现认不出dll的类，没法调用dll的方法。/类上面手动导入 using 他不会自动帮你补的。

11. **单步步过的调试，从哪里开始运行起来说明那里就是关键函数**

12. C#分析，在跟进的类中，对于public和private这些方法除了类名函数做方法名的初始化函数会运行之外，其其他众多方法除非被调用否则不用看。![](https://cdn-yg-zzbm.yun.qianxin.com/attack-forum/2022/08/attach-71d77ae624e91d36b39306bfc1724903d36a4d3f.png)

13. ![](https://cdn-yg-zzbm.yun.qianxin.com/attack-forum/2022/08/attach-9bbce8e47e01835cdc73c824002c5a8ef2f0dd63.png)

14. 现成loader：https://github.com/hexfati/SharpDllLoader，https://bbs.pediy.com/thread-260827.htm

15. 从内存中dump出来的PE需要重建IAT表的本质是原有的导入函数名都被替换成了真实的函数在内存中的地址，重建的操作将内存地址替换为函数名。

16. VirtualProtect函数是不是只给了代码段执行权限，所以按这个dump出来的不对。VirtualAlloc+Write函数dump出来的肯定是一个完整PE。

17. 型的加密方法，RSA+AES，非对称加密保证安全性，对称加密保证效率(多线程)。首先使用公钥P1对AES的密钥key1进行加密得到key2，再使用key2进行对称加密。解密时服务器下发私钥C1，得到公钥P1，继而得到key2，实现解密。

18. 不是具体哪个函数干了什么怎么干的，重要的是这个样本干了什么。不必拘泥于细节，重点在功能，这就是你在大多数报告里找不到技术细节的原因。

19. 空字符串的*md5*哈希值为d41d8cd98f00b204e9800998ecf8427e

20. 线程切换：先挂起当前正在运行的线程，再在目的线程入口点下断，最后恢复目的线程，不下断容易跑飞。

21. shellcode功能实际上大同小异，加载dll，建立通信。将shellcode加载进内存的操作也就那么几种，virtualalloc，virtualprotect。

22. 0号进程也叫IDLE进程，系统自动创建，运行在内核态。

23. CS木马：与0x2E异或解密出配置信息，使用http传输。

24. 恶意操作不会写在shellcode里，一般都是连接c2的核心代码，否则消耗太大。

25. x64dbg重命名分析出来的地址，选中地址，右键标签，标签xxxx。快捷键：alt+分号。

26. 脱不掉的壳，在虚拟机里跑起来，然后把进程dump下来，这时候一般恶意字符串都解密了。

27. winSDK10 GitHub 用于查rclsid

28. 针对eventlog的分析，还是推荐打成zip包传到观星上分析。

29. filesword：服务列表显示，包含服务的创建时间， mimikatz解析的内存密码，和解析的sam的hash（这种对默认是administrator激活状态，administrator是弱密码判断有帮助）。

30. 持久化控制：通过CLSID_TaskScheduler com对象注册服务来实现持久化控制。

31. 白加黑的参数没有意义，都是原始的参数。

32. **什么是stage(stageless)？**

    ```
    stage是无阶段的stager，可以直接理解成，stage是stager与它所请求的数据的 集合体。stage比stager更安全，但是体积更大。而且在内网穿透的时候基本只能用 stage，用stager会十分麻烦，stager是分段传输payload的，使用stager有时候 会导致目标无法上线。stage唯一的缺点是相比较而言体积比较大。
    ```

    **什么是stager？**

    ```
    stager其实是一段很简单的加载器，是socketedi协议请求的一段shellcode，它 的作用是向teamserver(C2)请求一段数据，这些数据前是个字节是shellcode的长 度，后面是shellcode。接收到数据后跳转到shellcode所在的内存处开始运行。
    ```
