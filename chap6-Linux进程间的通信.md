
Linux中的进程为了能在同一项任务上协调工作，它们彼此之间必须能够进行通信。
对于一个操作系统来说，进程间的通信是不可或缺的。
Linux支持多种不同方式的进程间通信机制，如信号、管道、FIFO和System V IPC机制，
其中System V IPC机制包括：信号量、消息队列和共享内存3种机制。
Linux下的这些进程间通信机制基本上是从UNIX平台上的进程间通信机制继承和发展而来的。


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



















































