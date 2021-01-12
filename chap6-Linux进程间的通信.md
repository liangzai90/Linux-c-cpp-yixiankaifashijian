
Linux中的进程为了能在同一项任务上协调工作，它们彼此之间必须能够进行通信。
对于一个操作系统来说，进程间的通信是不可或缺的。
Linux支持多种不同方式的进程间通信机制，如信号、管道、FIFO和System V IPC机制，
其中System V IPC机制包括：信号量、消息队列和共享内存3种机制。
Linux下的这些进程间通信机制基本上是从UNIX平台上的进程间通信机制继承和发展而来的。



进程使用系统调用kill将信号发送给一个进程或一组进程。
注意，这个系统调用kill不是杀死进程，而是一个进程发送信号给另一个进程。
其中，要求接收信号进程和发送信号进程的所有者相同，或者发送信号进程的所有者是超级用户。（权限问题）

Linux内核中并没有专门的机制来区分不同信号的相对优先级。
也就是说，当有多个信号在同一时刻发出时，进程可能会以任意的顺序接收到信号并进行处理。




- > 与信号相关的系统调用

系统调用kill()用来向一个进程或一个进程组发送一个信号，其中第一个参数决定信号发送的对象，
该系统调用声明如下：
```
#include <sys/types.h>
#incllude <signal.h>
int kill(pid_t pid, int sig);
```

- > 使用kill发送信号终止目标进程
```cpp
 
#include <sys/wait.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h> 
int main(void)
{
	pid_t childpid;
	int status;
	int retval;
    
	childpid = fork();
	if (-1 == childpid)
	{
		perror("fork()");
		exit(EXIT_FAILURE);
	}
	else if (0 == childpid)
	{
		puts("In child process");
		sleep(1000); 
		exit(EXIT_SUCCESS);
	}
	else
	{
		if (0 == (waitpid(childpid, &status, WNOHANG)))
		{
			retval = kill(childpid, SIGKILL);
            
			if (retval)
			{
				puts("kill failed.");
				perror("kill");
				waitpid(childpid, &status, 0);
			}
			else
			{
				printf("%d killed\n", childpid);
			}
            
		}
	}
    
	exit(EXIT_SUCCESS);
}

```


- > 系统调用sigaction 函数的简单使用

使用sigaction查询或设置信号处理方式

运行下面的程序，在另一个终端执行下面的命令：

kill -USR1  2540 

2540是进程id

```cpp
#include <stdio.h>
#include <unistd.h>
#include <signal.h>
#include <errno.h>

static void sig_usr(int signum)
{
	if (signum == SIGUSR1)
	{
		printf("SIGUSR1 received\n");
	}
	else if (signum == SIGUSR2)
	{
		printf("SIGUSR2 received\n");
	}
	else
	{
		printf("signal %d received\n", signum);
	}
}

int main(void)
{
	char buf[512];
	int  n;
	struct sigaction sa_usr;
	sa_usr.sa_flags = 0;
	sa_usr.sa_handler = sig_usr;   //信号处理函数
    
	sigaction(SIGUSR1, &sa_usr, NULL);
	sigaction(SIGUSR2, &sa_usr, NULL);
    
	printf("My PID is %d\n", getpid());
    
	while (1)
	{
		if ((n = read(STDIN_FILENO, buf, 511)) == -1)
		{
			if (errno == EINTR)
			{
				printf("read is interrupted by signal\n");
			}
		}
		else
		{
			buf[n] = '\0';
			printf("%d bytes read: %s\n", n, buf);
		}
	}
    
	return 0;
}
```


3.使用 sigprocmask 检测或更改信号屏蔽字
系统调用sigprocmask()可以检测或更改其信号屏蔽字。
一个进程的信号屏蔽字规定了当前阻塞而不能传递给该进程的信号集。

```cpp
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

void handler(int sig)
{
	printf("Deal SIGINT");  //SIGINT信号处理函数
}
int main()
{
	sigset_t newmask;
	sigset_t oldmask;
	sigset_t pendmask;

	struct sigaction act;
	act.sa_handler = handler;   
	sigemptyset(&act.sa_mask);
	act.sa_flags = 0;
	//信号捕捉函数，捕捉 Ctrl + C 
	sigaction(SIGINT, &act, 0);  
	sigemptyset(&newmask); 
	sigaddset(&newmask, SIGINT); 
	sigprocmask(SIG_BLOCK, &newmask, &oldmask); 
	sleep(5); 
	sigpending(&pendmask); 

	if (sigismember(&pendmask, SIGINT)) 
		printf(" SIGINT pending\n");
 
	sigprocmask(SIG_SETMASK, &oldmask, NULL); 

	 
	printf("SIGINT unblocked\n");
	sleep(5);   
	return (0);
}

```


使用sigpending检查是否有挂起的信号

使用signal设置信号处理程序

忽略 SIGINT 信号
```cpp
#include <stdio.h>
#include <signal.h>
int main(int argc, char *argv[]) 
{
	signal(SIGINT, SIG_IGN);
	while (1);
	return 0;
}
```
运行程序，多次执行Ctrl+C，程序没有退出，说明信号SIGINT被忽略了


自定义信号 SIGINT 的处理
```cpp

#include <stdio.h>
#include <signal.h>
typedef void(*signal_handler)(int);

void signal_handler_fun(int signum) {
	printf("catch signal %d\n", signum);
}

int main(int argc, char *argv[]) {
	signal(SIGINT, signal_handler_fun);
	while (1)
		;
	return 0;
}

```


### 管道
所谓管道，是指用于连接读进程和写进程，以实现他们之间通信的共享文件，故又称管道文件。
可以说管道是一种先进先出的方式，保存一定数量数据的特殊文件，而且管道一般是单向的。

父子进程使用管道通信
```cpp
#include <unistd.h>  
#include <string.h>  
#include <stdlib.h>  
#include <stdio.h>  
#include <sys/wait.h>  
      
void sys_err(const char *str)  
{  
	perror(str);  
	exit(1);  
}  
      
int main(void)  
{  
	pid_t pid;  
	char buf[1024];  
	int fd[2];  
	char p[] = "test for pipe\n";  
          
	if (pipe(fd) == -1)   
		sys_err("pipe");  
      
	pid = fork();  
	if (pid < 0) {  
		sys_err("fork err");  
	}
	else if (pid == 0) {  
		close(fd[1]);  
		printf("child process wait to read:\n");
		int len = read(fd[0], buf, sizeof(buf));  
		write(STDOUT_FILENO, buf, len);  
		close(fd[0]);  
	}
	else {  
		close(fd[0]);  
		write(fd[1], p, strlen(p));  
		wait(NULL);  
		close(fd[1]);  
	}  
          
	return 0;  
}  
```



























