
进程（Process）是操作系统结构的基础。
进程是一个具有独立功能的程序对某个数据集在处理机上的执行过程，
进程也是作为【资源分配】的一个基本单位。
Linux作为多个用户、多个任务的操作系统，必定支持多进程。
多进程是现代操作系统的基本特征。

- > 获取当前进程ID (getpid())
```cpp
#include <iostream>
#include <unistd.h>
using namespace std;

int main(int argc, char *argv[])
{
	pid_t pid = getpid();
	cout <<"pid="<<pid << endl;	 
	return 0;
}
```

- > 通过fork来创建子进程
```cpp
#include <iostream>
using namespace std;
#include <unistd.h>  
#include <stdio.h>   
int main()   
{   
	pid_t fpid;  
	int count = 0;  
	fpid = fork();   
	if (fpid < 0)   
		cout<<"failed to fork";   
	else if (fpid == 0) 
	{  
		cout<<"I am the child process, my pid is  "<<getpid()<<endl;   
		count++;  
	}  
	else 
	{  
		cout<<"I am the parent process, my pid is "<<getpid()<<endl; 
		cout << "fpid =" << fpid << endl;
		count++;  
	}  
	printf("count=%d\n", count);  
	return 0;  
}  
```
  输出内容：
```txt
I am the parent process, my pid is 2825
fpid =2826
count=1
I am the child process, my pid is  2826
count=1
```
fork调用的一个奇妙之处就是它仅仅被调用一次，却能够返回两次。
fork是把进程当前的情况复制一份，执行fork时，进程已经执行完了语句int count =0;，
fork只复制下一次要执行的代码到新的进程。


- > 使用exec创建进程

#include <unistd.h>
int execl(const char* path, const char* arg, ...);

参数path指向要执行的文件路径；
后面的参数（arg,...）代表执行该程序时传递的参数列表，
并且第1个被认为是argv[0](即path后面的参数被认为是argv[0])，第2个被认为是argv[1]....
最后一个参数必须是空指针 NULL结束。
函数成功时不返回值，失败则返回-1，失败原因在errno中，可通过perror()打印。

另外要注意，对于系统命令程序，比如pwd命令，argv[0]是必须要有的，但其值可以是一个无意义的字符串

使用execl执行不带选项的命令程序pwd
```cpp
#include <unistd.h>  

int main(int argc, char* argv[])
{
	execl("/bin/pwd", "asdfaf",  NULL);  
	
	return 0;
}
```

使用execl执行带选项的命令 程序ls
```cpp
 
#include <unistd.h>  
      
int main(void)  
{  
    //执行/bin目录下的ls  
    //第一个参数为程序名ls，第二个参数为-al,第三个参数为/etc/passwd  
	execl("/bin/ls", "ls", "-al", "/etc/passwd", NULL);  
	return 0;  
}  
```

使用execl执行我们的程序
```cpp

/////把生成的二进制文件命名为 mytest
/////把生成的二进制文件命名为 mytest
/////把生成的二进制文件命名为 mytest

#include <string.h>
using namespace std;
#include <iostream>
 
int main(int argc, char* argv[])
{
	int i;
	cout <<"argc=" << argc << endl; //打印下传进来的参数个数
	
	for(i=0;i<argc;i++)   //打印各个参数
		cout<<argv[i]<<endl;
	 
	if (argc == 2&&strcmp(argv[1], "-p")==0)  //判断是否如果带了参数-p
		cout << "will print all" << endl;
	else 
		cout << "will print little" << endl;
	
	cout << "my program over" << endl;
	return 0;
}



/////这个文件执行上面生成的二进制文件
#include <unistd.h>
using namespace std;
#include <iostream>
 
int main(int argc, char* argv[])
{
	cout <<"111---------------------------------"<<endl;

	//不带参数
	//execl("./mytest", NULL);

	cout <<"222==============================="<<endl;

	//带2个参数
	execl("./mytest", "henry", "-p", NULL);

	//特别注意，如果execl执行成功，下面这行代码是不会输出的！！！
	cout <<"333==============================="<<endl;

	return 0;
}

```

- > 函数execlp
execlp函数会从PATH环境变量所指的目录中查找符合参数file的文件名，找到后便执行该文件，
然后将第二个以后的参数当做该文件的argv[0]、argv[1]...，最后一个参数必须用空指针(NULL)结束。

execlp函数声明如下：

#include <unistd.h>
int execlp(const char* file, const char* arg, ...);

如果执行成功，则函数不返回，执行失败则直接返回-1，失败的原因存于errno中。


使用execlp执行不带选项的命令程序pwd
execlp的第一个参数直接用pwd这个命令程序即可，而不需要写出其全路径，
因为环境变量PATH中已经包含路径 /usr/bin 了
第2个参数必须有，可以是任意字符串.
```cpp
#include <unistd.h>  

int main(int argc, char* argv[])
{
	execlp("pwd", "",  NULL); 
	return 0;
}

```

- > 使用execlp执行我们的程序
```cpp

//把生成的文件命名为execlpTest（因为第2个可执行程序要执行execlpTest)
//把生成的文件命名为execlpTest（因为第2个可执行程序要执行execlpTest)
//把生成的文件命名为execlpTest（因为第2个可执行程序要执行execlpTest)

#include <string.h>
using namespace std;
#include <iostream>
 
int main(int argc, char* argv[])
{
	cout <<"execlp test  打印传入的参数个数，和各个参数"<<endl;
	int i;
	cout <<"argc=" << argc << endl; //打印下传进来的参数个数
	
	for(i=0;i<argc;i++)   //打印各个参数
		cout<<argv[i]<<endl;
}



///// 这里我们用的是相对路径，也可以把execlpTest这个文件的路径添加到系统目录下面，
//然后execlp的第一个参数就可以直接传文件名
#include <unistd.h>  
using namespace std;
#include <iostream>

int main(int argc, char* argv[])
{
	//execlp("./execlpTest", NULL); //不传任何参数给mytest
	//cout << "111------------------\n";//如果execlp执行成功，这一句不会执行到的。

	execlp("./execlpTest", "henry", "he", NULL);
	cout << "222------------------\n";//如果execlp执行成功，这一句不会执行到的。
	
	return 0;
}

```


### 进程调度
进程调度也就是处理机调度。在多道程序设计环境中，进程数往往多于处理机数，这将导致多个进程
对处理机资源的相互争夺。进程调度的任务是控制和协调进程对CPU的竞争，按照一定的调度算法
使某一就绪进程取得CPU的控制权，从而转为运行状态。

进程调度的功能主要包括：记录系统中所有进程的执行状态；根据一定的调度算法，从就绪队列
中选出一个进程来准备把处理机分配给它；将处理机分配给进程，进行上下文切换，把选中进程
的进程控制块内有关的现场信息（如程序状态字、通用寄存器等内容）送入处理器相应的寄存器中，
从而让它占用处理机运行。

进程的调度一般可以在下述情况下发生：
1.正在执行的进程运行完毕
2.正在执行的进程调用阻塞原语将自己阻塞，并来进入等待状态
3.执行中的进程提出I/O请求后被阻塞
4.正在执行的进程调用了P原语操作，因资源得不到满足而被阻塞；或者调用V原语操作释放了资源，从而激活了等待相应资源的进程队列
5.在分时系统中，时间片已经用完
6.就绪队列中的某个进程的优先级变得高于当前运行进程的优先级，从而引起进程的调度


## 常见的进程调度算法
1.先来先服务（FCFS）
2.时间片轮转法（RR）
3.优先级算法
4.多级反馈队列法


## 进程调度
自行补充这方面的知识

### 守护进程
守护进程（Daemon Process）是运行在后台的一种特殊进程。
它独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事情。
守护进程是一种很有用的进程。Linux中大多数服务器都是用守护进程实现的，
比如 Internet 服务器 inetd、Web服务器 httpd 等，另外还有常见的守护进程包括系统
日志进程 syslogd、数据库服务器 mysld等。

守护进程的特点：
1.守护进程都具有超级用户的权限
2.守护进程的父进程是init进程
3.守护进程都不用控制终端，其TTY列以 "?"表示，TPGID为 -1 
4.守护进程都是各自进程组合会话过程的唯一进程


查看守护进程：
```bash
ps axj
```


End