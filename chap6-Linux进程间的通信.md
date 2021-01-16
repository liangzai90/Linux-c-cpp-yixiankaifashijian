
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


read阻塞10秒后读数据
```cpp
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <fcntl.h> 

int main(void)
{
	int fds[2];
	if (pipe(fds) == -1) {
		perror("pipe error");
		exit(EXIT_FAILURE);
	}
	pid_t pid;
	pid = fork();
	if (pid == -1) {
		perror("fork error");
		exit(EXIT_FAILURE);
	}
	if (pid == 0) {
		close(fds[0]);//子进程关闭读端
		sleep(10);
		write(fds[1], "hello", 5);
		exit(EXIT_SUCCESS);
	}

	close(fds[1]);//父进程关闭写端
	char buf[10] = { 0 };
	read(fds[0], buf, 10);
	printf("receive datas = %s\n", buf);
	return 0;
}
```


## 消息队列
Linux提供了一组消息队列函数让我们使用消息队列。
```cpp
#include <sys/msg.h>
#include <sys/types.h>
#include <sys/ipc.h>

获取和设置消息队列的属性函数 msgctl
int msgctl(int msqid, int cmd, struct msqid_ds *buf);

创建和打开消息队列函数 msgget
int msgget(key_t key, int msgflg);

从消息队列中读取一条新消息的函数 msgrcv
int msgrcv(int msqid, void *msg_ptr, size_t msg_sz, long int msgtype, int msgflag);

将消息送入消息队列的函数 msgsnd
int msgsnd(int msqid, const void *msg_ptr, size_t msg_sz, int msgflg);

生成键值函数 ftok
key_t ftok(char* fname, int id);
```
系统建立IPC通信（如消息队列、共享内存）必须指定一个键值。


```cpp
// filename: test.cpp
#include <stdio.h>
#include <sys/sem.h>
#include <stdlib.h>
#include <iostream>
using namespace std;
int main()
{
	key_t semkey;
	if((semkey = ftok("./test", 123) < 0))
	{
		cout <<"ftok failed."<<endl;
		exit(EXIT_FAILURE);
	}

	cout <<"ftok ok, semkey ="<<semkey<<endl;

	return 0;
}

// g++ test.cpp -o test
```


### 消息队列的发送和接收

消息的接收
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
struct my_msg_st
{
    long int my_msg_type;
    char some_text[BUFSIZ];
};
int main()
{
    int running = 1;
    int msgid;
    struct my_msg_st some_data;
    long int msg_to_receive = 0;

		//设置消息队列：
    msgid = msgget((key_t)1234,0666|IPC_CREAT);
    if(msgid == -1)
    {
        fprintf(stderr,"msgget failed with error: %d\n", errno);
        exit(EXIT_FAILURE);
    }

    //接收消息队列中的消息直到遇到一个end消息。最后，消息队列被删除：
    while(running)
    {
		//阻塞方式等待接收消息
        if(msgrcv(msgid, (void *)&some_data, BUFSIZ, msg_to_receive, 0) == -1)
        {
            fprintf(stderr, "msgrcv failed with errno: %d\n", errno);
            exit(EXIT_FAILURE);
        }

        printf("You wrote: %s", some_data.some_text);
		//如果收到的是end，就退出循环
        if(strncmp(some_data.some_text, "end", 3)==0)
        {
            running = 0;
        }
    }
 
	//IPC_EMID 将队列从系统内核中删除
    if(msgctl(msgid, IPC_RMID, 0)==-1)
    {
        fprintf(stderr, "msgctl(IPC_RMID) failed\n");
        exit(EXIT_FAILURE);
    }
    exit(EXIT_SUCCESS);
}

```

消息的发送
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#define MAX_TEXT 512
struct my_msg_st
{
    long int my_msg_type;
    char some_text[MAX_TEXT];
};

 int main()
{
    int running = 1;
    struct my_msg_st some_data;
    int msgid;
    char buffer[BUFSIZ];

    msgid = msgget((key_t)1234, 0666|IPC_CREAT);//用一个整数作为键值
    if(msgid==-1)
    {
        fprintf(stderr,"msgget failed with errno: %d\n", errno);
        exit(EXIT_FAILURE);
    }
     while(running)
    {
        printf("Enter some text: ");
        fgets(buffer, BUFSIZ, stdin);
        some_data.my_msg_type = 1;
        strcpy(some_data.some_text, buffer);
        if(msgsnd(msgid, (void *)&some_data, MAX_TEXT, 0)==-1)
        {
            fprintf(stderr, "msgsnd failed\n");
            exit(EXIT_FAILURE);
        }
        if(strncmp(buffer, "end", 3) == 0)
        {
            running = 0;
        }
    }
     exit(EXIT_SUCCESS);
}
```
编译上面2个文件，分别运行 接收程序、发送程序


获取消息队列的属性
```cpp
#include <sys/types.h>
#include <sys/msg.h>
#include <unistd.h>
#include <ctime>
#include "stdio.h"
#include "errno.h"
void msg_stat(int,struct msqid_ds );
main()
{
	int gflags,sflags,rflags;
	key_t key;
	int msgid;
	int reval;
	struct msgsbuf{
			int mtype;
			char mtext[1];
		}msg_sbuf;
	struct msgmbuf
		{
		int mtype;
		char mtext[10];
		}msg_rbuf;
	struct msqid_ds msg_ginfo,msg_sinfo;
	char  msgpath[]="./test";

	key=ftok(msgpath,'b');
	gflags=IPC_CREAT|IPC_EXCL;
	msgid=msgget(key,gflags|00666);
	if(msgid==-1)
	{
		printf("msg create error\n");
	}
	//创建一个消息队列后，输出消息队列缺省属性
	msg_stat(msgid,msg_ginfo);
	sflags=IPC_NOWAIT;
	msg_sbuf.mtype=10;
	msg_sbuf.mtext[0]='a';
	reval=msgsnd(msgid,&msg_sbuf,sizeof(msg_sbuf.mtext),sflags);
	if(reval==-1)
	{
		printf("message send error\n");
	}
	//发送一个消息后，输出消息队列属性
	msg_stat(msgid,msg_ginfo);

	 
	reval=msgctl(msgid,IPC_RMID,NULL);//删除消息队列
	if(reval==-1)
	{
		printf("unlink msg queue error\n");
	}
}
void msg_stat(int msgid,struct msqid_ds msg_info)
{
	int reval;
	sleep(1);//只是为了后面输出时间的方便
	reval=msgctl(msgid,IPC_STAT,&msg_info);
	if(reval==-1)
	{
		printf("get msg info error\n");
	}
	printf("\n");
	printf("current number of bytes on queue is %d\n",msg_info.msg_cbytes);
	printf("number of messages in queue is %d\n",msg_info.msg_qnum);
	printf("max number of bytes on queue is %d\n",msg_info.msg_qbytes);
	//每个消息队列的容量（字节数）都有限制MSGMNB，值的大小因系统而异。在创建新的消息队列时，//msg_qbytes的缺省值就是MSGMNB
	printf("pid of last msgsnd is %d\n",msg_info.msg_lspid);
	printf("pid of last msgrcv is %d\n",msg_info.msg_lrpid);
	printf("last msgsnd time is %s", ctime(&(msg_info.msg_stime)));
	printf("last msgrcv time is %s", ctime(&(msg_info.msg_rtime)));
	printf("last change time is %s", ctime(&(msg_info.msg_ctime)));
	printf("msg uid is %d\n",msg_info.msg_perm.uid);
	printf("msg gid is %d\n",msg_info.msg_perm.gid);
}

```








