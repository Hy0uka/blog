+++
title = '几种dll注入技术的正向实现'

description = "DLL注入是将代码加载到目标进程而不是只是内存"

tags = [ "技术" ]

date = 2024-06-26T18:28:27+08:00

+++

#### CreateRemoteThread注入

​	这种注入方式可以说是最常用的注入方式了，它的核心就是调用Windows提供的CreateRemoteThread函数。该函数可以在其他的进程空间中创建一个新的线程进行执行，该函数在文档中的定义如下

```c
HANDLE WINAPI CreateRemoteThread(
  __in   HANDLE hProcess,
  __in   LPSECURITY_ATTRIBUTES lpThreadAttributes,
  __in   SIZE_T dwStackSize,
  __in   LPTHREAD_START_ROUTINE lpStartAddress,
  __in   LPVOID lpParameter,
  __in   DWORD dwCreationFlags,
  __out  LPDWORD lpThreadId);
```

| **参数**           | **说明**                                                 |
| ------------------ | -------------------------------------------------------- |
| **hProcess**       | **要创建线程的进程句柄**                                 |
| lpThreadAttributes | 新线程的安全描述符                                       |
| dwStackSize        | 堆栈起始大小，为0表示默认大小                            |
| **lpStartAddress** | **表示要运行线程的起始地址**                             |
| **lpParameter**    | **保存要传递给线程参数的地址**                           |
| dwCreationFlags    | 控制线程创建的标志，为0表示创建后立即执行                |
| lpThreadId         | 指向接收线程标识符变量的指针。为NULL表示不返回线程标识符 |

​	那也就是说只要我们指定了一个地址给lpStartAddress，那么我们就可以在其他进程中创建一个线程来执行程序。而再看加载DLL的LoadLibrary函数在文档中的定义如下

`HMODULE` `WINAPI LoadLibrary(__in ``LPCTSTR` `lpFileName);`

​	可以看到，这个函数同样也只需要一个参数，这个参数是一个地址，而这个地址中保存的是我们要加载的DLL的名称的字符串。

​	根据这些，不难想到，只要我们可以获取新进程中的LoadLibrary函数的地址以及包含有要加载的DLL的字符串的地址就可以通过CreateRemoteThread函数来成功开起一个线程执行LoadLibrary函数来加载我们的DLL。

​	那么现在的问题就是如何获得LoadLibrary函数的地址以及保存有要加载的DLL路径的字符串的地址。

​	对于LoadLibrary函数，由于它是在常用的系统DLL，也就是KERNEL32.dll中，所以这个DLL是可以按照它的ImageBase成功装载到每个进程的空间中。这样的话Kernel32.dll在每个进程中的起始地址是一样的，那么LoadLibrary函数的地址也就会一样。那么我们就可以在本进程中查找LoadLibrary函数的地址，并且完全可以相信，在要注入DLL的进程中LoadLibrary的地址也是这个。

​	至于DLL名称的字符串，我们可以通过在进程中申请一块可以将DLL完整路径写入的内存，并在这个内存中将DLL的完整路径写入，将写入到注入进程DLL完整路径的内存地址作为参数就可以实现进程的注入。

#### RtlCreateUserThread创建用户线程

 RtlCreateUserThread是CreateRemoteThread的底层实现，所以使用RtlCreateUserThread的原理是和使用CreateRemoteThread的原理是一样的。唯一的区别是使用CreateRemoteThread写入目标进程的是Dll的路径，而RtlCreateUserThread写入的是一段shellcode。因为直接调用了RtlCreateUserThread创建线程，绕过了win7/vista对这方面的检测所以可以注入到系统进程中(比如winlogon.exe)，当然此方法也有缺陷，因为没有和csrss通讯，可能在线程体里面调用某些api函数可能会失败。还有需要注意的是因为RtlCreateUserThread没有经过系统封装所以线程体必须自己调用ExitThread来结束线程。

#### QueueUserApc/NtQueueAPCThread APC注入法

APC是Asynchronous Procedure Call的缩写，是一种软中断机制。当一个线程从等待状态（线程调用SleepEx,SignalObjectAndWait,MsgWaitForMultiple ObjectsEx,WaitForMultipleObjectsEx函数时会进入可唤醒状态）中苏醒时，会检测有没有APC交付给自己。如果有，就会执行APC过程。APC有两种形式，由系统产生的APC称为内核模式APC，由应用程序产生的APC称为用户模式APC。	

​	对于用户模式的APC队列，当线程处在可警告状态时，就会执行这些APC函数。而要往APC队列中增加APC函数，需要通过QueueUserAPC函数来实现，这个函数在文档中的定义如下

```c
DWORD WINAPI QueueUserAPC(
  __in  PAPCFUNC pfnAPC,
  __in  HANDLE hThread,
  __in  ULONG_PTR dwData);
```

| **参数** | **说明**                            |
| -------- | ----------------------------------- |
| pfnAPC   | 当满足条件时，要执行的APC函数的地址 |
| hThread  | 指定增加APC函数的线程句柄           |
| dwData   | 要执行的APC函数参数地址             |

可以看到pfnAPC和dwData这两个参数和CreateRemoteThread中的lpStartAddress和lpParameter的作用是一样的。**不过这里是对线程进行操作，一个进程有多个线程。所以为了确保程序正确运行，所以需要遍历所有线程，查看是否是要注入的进程的线程，依次获得句柄插入APC函数。**为了增加调用机会，应向所有线程插入APC，所以一般调用CreateToolhelp32Snapshot

```c
BOOL InjectDll(DWORD dwPid, CHAR szDllName[])
{
    BOOL bRet = TRUE;
    HANDLE hProcess = NULL, hThread = NULL, hSnap = NULL;
    HMODULE hKernel32 = NULL;
    DWORD dwSize = 0;
    PVOID pDllPathAddr = NULL;
    PVOID pLoadLibraryAddr = NULL;
    THREADENTRY32 te32 = { 0 };
 
    // 打开注入进程，获取进程句柄
    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPid);
    if (NULL == hProcess)
    {
        ShowError("OpenProcess");
        bRet = FALSE;
        goto exit;
    }
 
    // 在注入进程中申请可以容纳DLL完成路径名的内存空间
    dwSize = 1 + strlen(szDllName);
    pDllPathAddr = VirtualAllocEx(hProcess, NULL, dwSize, MEM_COMMIT, PAGE_READWRITE);
    if (!pDllPathAddr)
    {
        ShowError("VirtualAllocEx");
        bRet = FALSE;
        goto exit;
    }
 
    // 把DLL完成路径名写入进程中
    if (!WriteProcessMemory(hProcess, pDllPathAddr, szDllName, dwSize, NULL))
    {
        ShowError("WriteProcessMemory");
        bRet = FALSE;
        goto exit;
    }
 
 
    hKernel32 = LoadLibrary("kernel32.dll");
    if (hKernel32 == NULL)
    {
        ShowError("LoadLibrary");
        bRet = FALSE;
        goto exit;
    }
 
    // 获取LoadLibraryA函数地址
    pLoadLibraryAddr = GetProcAddress(hKernel32, "LoadLibraryA");
    if (pLoadLibraryAddr == NULL)
    {
        ShowError("GetProcAddress");
        bRet = FALSE;
        goto exit;
    }
 
    //获得线程快照
    hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);
    if (!hSnap)
    {
        ShowError("CreateToolhelp32Snapshot");
        bRet = FALSE;
        goto exit;
    }
 
    //遍历线程
    te32.dwSize = sizeof(te32);
    if (Thread32First(hSnap, &te32))
    {
        do
        {
            //这个线程的进程ID是不是要注入的进程的PID
            if (te32.th32OwnerProcessID == dwPid)
            {
                hThread = OpenThread(THREAD_ALL_ACCESS, FALSE, te32.th32ThreadID);
                if (hThread)
                {
                    QueueUserAPC((PAPCFUNC)pLoadLibraryAddr, hThread, (ULONG_PTR)pDllPathAddr);
                    CloseHandle(hThread);
                    hThread = NULL;
                }
                else
                {
                    ShowError("OpenThread");
                    bRet = FALSE;
                    goto exit;
                }
            }
        } while (Thread32Next(hSnap, &te32));
    }
exit:
    if (hKernel32) FreeLibrary(hKernel32);
    if (hProcess) CloseHandle(hProcess);
    if (hThread) CloseHandle(hThread);
    return bRet;
}
```

在实际使用中，由于条件苛刻，能够成功利用的机会并不多。

#### 反射型DLL注入

https://www.cnblogs.com/fdxsec/p/18300826

https://yaoyue123.github.io/2021/01/31/Windows-Reflective-dllinject/#GetReflectiveLoaderOffset%E7%9A%84%E5%AE%9E%E7%8E%B0

反射 DLL 注入又称 RDI，与其他注入不同的是，其不需要使用LoadLibrary这一函数，而是自己来实现整个装载过程。我们可以为待注入的DLL添加一个导出函数，ReflectiveLoader，这个函数实现的功能就是装载它自身。那么我们只需要将这个DLL文件写入目标进程的虚拟空间中，然后通过DLL的导出表找到这个ReflectiveLoader并调用它，我们的任务就完成了。
那么我们的任务就到了如何编写这个函数上面了，由于这个函数执行的时候 DLL 还没有被加载，这个函数的编写也会受到诸多限制，比如说无法正常使用全局变量，还有我们的函数必须编写成与地址无关的函数，就像 shellcode 那样，无论加载到了内存中的哪一个位置都要保证成功加载。

优点：反射式注入方式并没有通过LoadLibrary等API来完成DLL的装载，DLL并没有在操作系统中”注册”自己的存在，因此ProcessExplorer等软件也无法检测出进程加载了该DLL。利用解密磁盘上加密的文件、网络传输等方式避免文件落地，DLL文件可以不一定是本地文件，可来自网络等，总之将数据写到缓冲区即可。由于它没有通过系统API对DLL进行装载，操作系统无从得知被注入进程装载了该DLL，所以检测软件也无法检测它。同时，由于操作流程和一般的注入方式不同，反射式DLL注入被安全软件拦截的概率也会比一般的注入方式低。

反射式注入流程如下：

1、读入原始DLL文件至内存缓冲区； 2、解析DLL标头并获取SizeOfImage； 3、为DLL分配新的内存空间，大小为SizeOfImage； 4、将DLL标头和PE节复制到步骤3中分配的内存空间； 5、执行重定位； 6、加载DLL导入的库； 7、解析导入地址表（IAT）； 8、调用DLL的DLL_PROCESS_ATTACH；

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/blog/2025/06/dab6256707157075dc16ced5da90d1ff.png)

#### 全局钩子注入

Windows中大部分的应用程序都是基于消息机制的，它们都有一个消息过程函数比如`SetWindowsHookEx`函数

，根据不同的消息完成不同的功能。

消息钩子是windows提供的一种消息过滤和预处理机制，可以用来截获和监视系统中的消息。

按照钩子作用范围不同，又可以分为局部钩子和全局钩子。局部钩子是针对某个线程的，而全局钩子是作用于整个系统的基于消息的应用。全局钩子需要使用DLL文件，在DLL文件中实现相应的钩子函数。

```c
HHOOK WINAPI SetWindowsHookExW(
_In_ int idHook,
_In_ HOOKPROC lpfn,
_In_opt_ HINSTANCE hmod,
_In_ DWORD dwThreadId);
```

`idHook` 要安装的钩子(HOOK)的类型，它决定了`HOOKPROC`被调用的时机。

`lpfn`指向钩子回调函数的指针。如果最后一个参数`dwThreadId`为0或者是其它进程创建的线程标识符，则`lpfn`参数必须指向DLL中的钩子回调函数，即`HOOKPROC`函数必须在DLL中实现。否则，`lpfn` 可以指向与当前进程相关联的代码中的钩子过程。

`hmod`包含由`lpfn`参数指向的钩子回调函数的`DLL`句柄。如果dwThreadId参数指定由当前进程创建线程，并且钩子回调函数位于当前进程关联的代码中，则`hmod`参数必须设置为`NULL`。

dwThreadId与钩子程序关联的线程标识符(指定要HOOK的线程 ID)。如果此参数为0，则钩子过程与系统中所有线程相关联，即全局消息钩子。

**通过`SetWindowsHookExW`函数安装一个用于过滤特定类型消息的钩子函数(钩子`WH_GETMESSAGE`可以立即被触发，而某些类型的钩子需要在处理指定类型的消息时才能被触发)，当其它进程中产生了我们过滤的消息，就会调用`HOOKPROC`，如果发现目标`DLL`(`HOOKPROC`实现的`DLL`)尚未加载,就会使用`KeUserModeCallback`函数回调`User32.dll`的`__ClientLoadLibrary()` 函数，由`User32.dll`把这个`DLL`加载到目标进程中(低级键盘和鼠标钩子的加载有所不同)，从而实现`DLL`注入其它进程的目的。**

#### SetThreadContext法挂起线程注入

正在执行的新线程可以被SuspendThread函数暂停执行，然后系统就会把此时线程的上下文环境保存下来。当使用ResumeThread函数恢复线程的执行时，系统会把之前保存的上下文环境恢复，使得线程从之前保存的EIP开始执行。在注入DLL时，可以将目标线程暂停，写入加载DLL的ShellCode，就能实现注入的效果。

1. 枚举目标进程中的线程，获得线程ID：CreateToolhelp32Snapshot、Thread32First
2. 打开进程和线程，暂停线程：OpenProcess、OpenThread、SuspendThread
3. 获取线程的Context并保存EIP：GetThreadContext，uEIP=Context.Eip
4. 申请内存，写入加载DLL的shellcode
5. 设置新的CONTEXT并恢复线程的执行：SetThreadCContext、ResumeThread

#### 挂起进程注入

CreateProcess注入方法之一，CREATE_SUSPENDED以挂起的方式打开进程，后面方法与挂起线程注入相似。

#### Process Hollowing

https://idiotc4t.com/code-and-dll-process-injection/process-hollowing

1. 创建 `notepad.exe`，使用 `CREATE_SUSPENDED` 启动。
2. 调用 `ZwUnmapViewOfSection` 解除原始映像。
3. 分配内存，写入恶意 PE 映像。
4. 设置入口地址（`SetThreadContext`）。
5. 恢复线程（`ResumeThread`），执行恶意程序。

→ 目标看起来是 `notepad.exe`，但实际运行的是你的恶意 PE 程序。

**SetThreadContext + DLL 注入流程：**

1. 附加或创建目标进程。
2. 分配内存并写入 DLL 路径。
3. 获取目标线程的线程上下文。
4. 设置 `EIP/RIP` 指向 `LoadLibraryA` 的地址，并传入 DLL 路径参数。
5. 恢复线程，目标进程加载你注入的 DLL。

→ 目标还是它自己，只是被“外挂”地加载了一个 DLL。
