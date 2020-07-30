### 记录书中用到的若干Linux命令


- > 查看网卡信息
```bash
[henry@localhost ~]$ ifconfig 
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.119.131  netmask 255.255.255.0  broadcast 192.168.119.255
        inet6 fe80::72c8:e8ce:298b:205d  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:9c:bc:97  txqueuelen 1000  (Ethernet)
        RX packets 2415  bytes 171293 (167.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 794  bytes 95771 (93.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 68  bytes 5920 (5.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 68  bytes 5920 (5.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[henry@localhost ~]$ 
```

- > 设置网卡信息
```bash
ens33  就是我们通过 ifconfig 查询到的网卡名称（不同机器名称不同）

[henry@localhost ~]$  vi /etc/sysconfig/network-scripts/ifcfg-ens33 
```

- > 查看网卡信息
```bash
[henry@localhost ~]$ cat /etc/sysconfig/network-scripts/ifcfg-ens33 
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="05d55e32-4139-4e4a-9e75-b3b85486edbc"
DEVICE="ens33"
ONBOOT="yes"
[henry@localhost ~]$ 
```

- > 重启网络服务
```bash
[henry@localhost ~]$ service network restart
Restarting network (via systemctl):  ==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to manage system services or units.
Authenticating as: henry
Password: 
==== AUTHENTICATION COMPLETE ===
                                                           [  OK  ]
[henry@localhost ~]$ 
```



- > windows下vscode 通过ssh 访问linux(CentOS7)的文件

```txt
1.windows下安装vscode
2.vscode 安装 Remote - SSH 插件

顺便把下面几个插件也安装了
VS Code CPP 相关插件
C/C++
C++ Intellisense
ESLint
vscode-icons
Remote Development
Better Comments

3.linux开启22端口
sudo firewall-cmd --zone=public --add-port=22/tcp --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --query-port=22/tcp

4.Linux安装 ssh server 
sudo yum install openssh-server && sudo systemctl start sshd.service && sudo systemctl enable sshd.service

5.windows 安装 ssh client
以管理员身份启动PowerShell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

6.windows 生成公钥和私钥
ssh-keygen -t rsa -b 4096
不需要输入啥密码，都是回车（如果你的目录下面已经存在其他的ssh私钥，你可以考虑在.ssh目录下面新建文件夹，将新生成的私钥指定到新的文件夹下面）

7.CentOS7 添加authorized_keys文件
mkdir /home/henry/.ssh
vi /home/henry/.ssh/authorized_keys
把 公钥里面的内容，添加到 authorized_keys 文件（如果authorized_keys之前不存在就先创建）

8.windows 远程连接 （用户名@IP地址）的格式，测试是否OK
C:\Users\Administrator>ssh henry@192.168.163.132 -p 22
henry@192.168.163.132's password:
Last login: Sun May 31 23:26:34 2020 from 192.168.163.1
[henry@localhost ~]$ ls
cppProject Desktop Documents Downloads Music packageRoot Pictures Public Templates Videos
[henry@localhost ~]$

9.打开vscode，点击左侧的 Remote Exploer
配置 config 文件，文件内容如下：
填写主机名称和登录用户名。

# Read more about SSH config files: https://linux.die.net/man/5/ssh_config
Host centos7.5
    HostName 192.168.119.131
    User henry

	
然后开启远程访问，需要2次输入密码。

```

- > GCC 编译器的使用
#### gcc 对C语言的编译过程

```txt
gcc对c/c++语言的编译过程可分为4个阶段：
预处理（Preprocess）、编译（Compilation）、汇编（Assembly）和链接（Linking）

/////// test.c
#include <stdio.h>
int main()
{
        char sz = "Hello, world!\r\n";
        printf("%s", sz);
        fflush(stdout);
        return0;
}


1.预处理
预处理就是对源程序中的伪指令（以#开头的指令）和特殊符号进行处理的过程。
伪指令包括宏定义指令、条件编译指令和头文件包含指令。
gcc对C源文件进行预处理后会输出.i文件。

gcc -E test.c -o test.i

选项 "-E" 告诉gcc只进行预处理；

"test.c"是C源文件；
"-o"用于指定要生成的结果文件，后面跟的就是结果文件名字。
选项"-o"中的o是output的意思，不是目标的意思。
-o后面跟的是要输出的结果的文件名称，结果文件可能是预处理文件、汇编文件、目标文件或者最终的可执行文件。



2.编译
编译过程就是把预处理完的文件进行一系列词法分析、语法分析、语义分析及优化后生成相应的汇编代码文件。
在使用gcc进行编译时，默认情况下，不输出这个汇编代码的文件。
如果需要，可以在编译时指定 -S 选项，这样就会输出同名的汇编语言文件。
gcc对C源文件编译后生成的汇编代码文件是.s文件。我们对上面的 test.i 进行编译：

gcc -S test.i -o test.s

选项 "-S" 告诉gcc只进行到编译阶段；



3.汇编
汇编就是讲汇编代码转为机器可以执行的二进制代码，每一个汇编语句几乎都对应一条机器指令。
汇编相对于编译过程比较简单，根据汇编指令和机器指令的对照表一一翻译即可。
gcc生成的二进制代码文件为后缀名为.o的文件。我们对上面的test.s进行汇编

gcc -c test.s -o test.o

选项 "-c" 告诉gcc只进行到汇编处理为止；
"test.s"是进行汇编的源文件；
"-o"用于指定要生成的结果文件，后面跟的就是结果文件名字；
这里输出的结果文件为目标文件 test.o，是一个二进制文件（不是文本文件），在windows上通常就是obj文件。
test.o 是二进制文件，可以用命令  hexdump 查看

hexdump test.o 



4.链接
在成功汇编之后，就进入了链接阶段。链接主要是为了解决多个文件之间符号引用的问题（symbol resolution）。
编译时编译器只对单个文件进行处理，如果该文件里面需要引用到其他文件中的符号（例如全局变量或者某个函数库中的函数），
那么这时在这个文件中该符号的地址是没法确定的，只能等链接器把所有的目标文件连接到一起才能确定最终的地址，最终生成可执行的文件。
当所有的目标文件都生成之后，gcc就在内部调用链接器 ld 来完成链接工作。
在链接阶段，所有的目标文件被安排在可执行程序中的恰当位置。
值得注意的是，在Linux系统中，可执行文件没有统一的后缀，系统从文件的属性来区分可执行文件和不可执行文件。

通过这个命令查看 当前系统的 glibc 版本
/lib64/libc.so.6

glibc 是GNU组织对C的标准实现库，是操作系统UNIX/Linux的基石之一。
微软也有自己的C标准实现库，叫 msvcrt；
嵌入式行业里还常用 uClibc，是一个迷你版的C标准实现库。

对上面生成的 test.o 进行链接：
gcc test.o -o test

这里只有一个目标文件，如果有多个目标文件（.o文件）可以写在一起，每两个目标文件之间加空格，
比如 gcc test1.o test2.o test3.o -o test 

最后，讲最终生成可执行文件 test 。

```

- > 计算文件的md5
```shell
[heliang@VM_0_10_centos ~]$ md5sum HelloWebWorld.html 
7ff69dfc537f293b4ce84f094dd0f191  HelloWebWorld.html
[heliang@VM_0_10_centos ~]$ 

```

- > 创建目录
```
-p 的意思是如果父目录不存在，就先创建父目录，再创建子目录。

[heliang@VM_0_10_centos ~]$ mkdir  abc/def 
mkdir: cannot create directory ‘abc/def’: No such file or directory
[heliang@VM_0_10_centos ~]$ mkdir -p abc/def 
[heliang@VM_0_10_centos ~]$ ls
abc          
```


- > 双引号包含时的搜索次序
```
1.当前 工作目录
2.-I 包含 目录
3./usr/local/include 目录
4./usr/include 目录
```

- > gcc常见选项
```
-x  
指定要编译的源文件是什么语言文件（不用根据后缀去判断）
gcc -x c test.pig 

-o
指定要生成的结果，后面跟的就是结果文件的名字。
o 是 output 的意思，不是目标的意思。

-c
告诉gcc对源文件进行编译和汇编，但不进行链接
gcc -c test.cpp

-I
用来指定头文件所在文件夹路径
gcc test.cpp -I /henry/include -o test

-include
gcc命令中也能包含头文件。
有时候源码里没有找到 #include <xxx.h>，但是<xxx.h>的内容确实被引用了。
在gcc编译时，通过-include来保护xxx.h。

gcc [srcfile] -include [headfile]
gcc test.cpp -include /henry/include/test.h -o test

-Wall
显示所有警告信息
gcc test.cpp -Wall -o test

-g
可以产生供gdb调试用的可执行文件。
gcc test.cpp -g -o test

-pg
能产生供 gprof 剖析用的可执行文件。
gprof 是 Linux 下对 C++ 程序进行性能分析的工具。

-l
（小写的L）可以用来链接共享库（动态链接库）
gcc test.cpp -lstdc++ -o test

stdc++是C++标准库的名字，它和l之间没空格。
```

- > gdb调试器的使用
```
gdb有4个方面的功能：
1）启动你的程序，可以按照自定义的要求随心所欲地运行程序。
2）可让被调试的程序在你所指定的调试的断点处停住（断电可以是表达式）。
3）当程序被停住时，可以检查此时你的程序中所发生的事，比如查看某个变量值、查看内存堆栈内容等。
4）动态改变你的程序的执行环境。

如果要用gdb调试，编译的时候需要使用 -g 

如果编译的时候，没加-g选项，用gdb调试，会提示：(no debugging symbols found)

gdb 的启动方式：

gdb <program> 

gdb <program> core

gdb <program> <PID>


gdb调试test文件

gdb 
file test

直接调试test文件
gdb test

list lineNumber
显示指定行前后的源码内容

list n1,n2
显示n1行到n2行的源码内容

list functionName
显示该函数的上下10行内容

run 
使用run命令可以在gdb中运行调试中的程序。run命令可以跟一个或多个参数

run boy girl
会把 boy,girl作为参数传递到main函数入口


break 命令设置断点

1)根据行号设置断点，比如： (gdb) break linenumber
2)根据函数名设置断点，比如 (gdb) break funcname
3)执行非当前源文件的某行或某函数时停止执行，比如： 
	(gdb) break filename:linenumber  
	或者
	(gdb) break filename:funcname 
4)根据条件停止执行程序，比如： 
	(gdb) break linenum if expr
	或者
	(gdb) break funcname if expr


continue  断点处继续执行
简写为 c 

```






















