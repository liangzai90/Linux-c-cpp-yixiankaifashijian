
Linux�еĽ���Ϊ������ͬһ��������Э�����������Ǳ˴�֮������ܹ�����ͨ�š�
����һ������ϵͳ��˵�����̼��ͨ���ǲ��ɻ�ȱ�ġ�
Linux֧�ֶ��ֲ�ͬ��ʽ�Ľ��̼�ͨ�Ż��ƣ����źš��ܵ���FIFO��System V IPC���ƣ�
����System V IPC���ư������ź�������Ϣ���к͹����ڴ�3�ֻ��ơ�
Linux�µ���Щ���̼�ͨ�Ż��ƻ������Ǵ�UNIXƽ̨�ϵĽ��̼�ͨ�Ż��Ƽ̳кͷ�չ�����ġ�



����ʹ��ϵͳ����kill���źŷ��͸�һ�����̻�һ����̡�
ע�⣬���ϵͳ����kill����ɱ�����̣�����һ�����̷����źŸ���һ�����̡�
���У�Ҫ������źŽ��̺ͷ����źŽ��̵���������ͬ�����߷����źŽ��̵��������ǳ����û�����Ȩ�����⣩

Linux�ں��в�û��ר�ŵĻ��������ֲ�ͬ�źŵ�������ȼ���
Ҳ����˵�����ж���ź���ͬһʱ�̷���ʱ�����̿��ܻ��������˳����յ��źŲ����д���




- > ���ź���ص�ϵͳ����

ϵͳ����kill()������һ�����̻�һ�������鷢��һ���źţ����е�һ�����������źŷ��͵Ķ���
��ϵͳ�����������£�
```
#include <sys/types.h>
#incllude <signal.h>
int kill(pid_t pid, int sig);
```

- > ʹ��kill�����ź���ֹĿ�����
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


- > ϵͳ����sigaction �����ļ�ʹ��

ʹ��sigaction��ѯ�������źŴ���ʽ

��������ĳ�������һ���ն�ִ����������

kill -USR1  2540 

2540�ǽ���id

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
	sa_usr.sa_handler = sig_usr;   //�źŴ�����
    
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


3.ʹ�� sigprocmask ��������ź�������
ϵͳ����sigprocmask()���Լ���������ź������֡�
һ�����̵��ź������ֹ涨�˵�ǰ���������ܴ��ݸ��ý��̵��źż���

```cpp
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

void handler(int sig)
{
	printf("Deal SIGINT");  //SIGINT�źŴ�����
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
	//�źŲ�׽��������׽ Ctrl + C 
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


ʹ��sigpending����Ƿ��й�����ź�

ʹ��signal�����źŴ������

���� SIGINT �ź�
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
���г��򣬶��ִ��Ctrl+C������û���˳���˵���ź�SIGINT��������


�Զ����ź� SIGINT �Ĵ���
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


### �ܵ�
��ν�ܵ�����ָ�������Ӷ����̺�д���̣���ʵ������֮��ͨ�ŵĹ����ļ������ֳƹܵ��ļ���
����˵�ܵ���һ���Ƚ��ȳ��ķ�ʽ������һ���������ݵ������ļ������ҹܵ�һ���ǵ���ġ�

���ӽ���ʹ�ùܵ�ͨ��
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



























