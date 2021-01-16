
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


read����10��������
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
		close(fds[0]);//�ӽ��̹رն���
		sleep(10);
		write(fds[1], "hello", 5);
		exit(EXIT_SUCCESS);
	}

	close(fds[1]);//�����̹ر�д��
	char buf[10] = { 0 };
	read(fds[0], buf, 10);
	printf("receive datas = %s\n", buf);
	return 0;
}
```


## ��Ϣ����
Linux�ṩ��һ����Ϣ���к���������ʹ����Ϣ���С�
```cpp
#include <sys/msg.h>
#include <sys/types.h>
#include <sys/ipc.h>

��ȡ��������Ϣ���е����Ժ��� msgctl
int msgctl(int msqid, int cmd, struct msqid_ds *buf);

�����ʹ���Ϣ���к��� msgget
int msgget(key_t key, int msgflg);

����Ϣ�����ж�ȡһ������Ϣ�ĺ��� msgrcv
int msgrcv(int msqid, void *msg_ptr, size_t msg_sz, long int msgtype, int msgflag);

����Ϣ������Ϣ���еĺ��� msgsnd
int msgsnd(int msqid, const void *msg_ptr, size_t msg_sz, int msgflg);

���ɼ�ֵ���� ftok
key_t ftok(char* fname, int id);
```
ϵͳ����IPCͨ�ţ�����Ϣ���С������ڴ棩����ָ��һ����ֵ��


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


### ��Ϣ���еķ��ͺͽ���

��Ϣ�Ľ���
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

		//������Ϣ���У�
    msgid = msgget((key_t)1234,0666|IPC_CREAT);
    if(msgid == -1)
    {
        fprintf(stderr,"msgget failed with error: %d\n", errno);
        exit(EXIT_FAILURE);
    }

    //������Ϣ�����е���Ϣֱ������һ��end��Ϣ�������Ϣ���б�ɾ����
    while(running)
    {
		//������ʽ�ȴ�������Ϣ
        if(msgrcv(msgid, (void *)&some_data, BUFSIZ, msg_to_receive, 0) == -1)
        {
            fprintf(stderr, "msgrcv failed with errno: %d\n", errno);
            exit(EXIT_FAILURE);
        }

        printf("You wrote: %s", some_data.some_text);
		//����յ�����end�����˳�ѭ��
        if(strncmp(some_data.some_text, "end", 3)==0)
        {
            running = 0;
        }
    }
 
	//IPC_EMID �����д�ϵͳ�ں���ɾ��
    if(msgctl(msgid, IPC_RMID, 0)==-1)
    {
        fprintf(stderr, "msgctl(IPC_RMID) failed\n");
        exit(EXIT_FAILURE);
    }
    exit(EXIT_SUCCESS);
}

```

��Ϣ�ķ���
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

    msgid = msgget((key_t)1234, 0666|IPC_CREAT);//��һ��������Ϊ��ֵ
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
��������2���ļ����ֱ����� ���ճ��򡢷��ͳ���


��ȡ��Ϣ���е�����
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
	//����һ����Ϣ���к������Ϣ����ȱʡ����
	msg_stat(msgid,msg_ginfo);
	sflags=IPC_NOWAIT;
	msg_sbuf.mtype=10;
	msg_sbuf.mtext[0]='a';
	reval=msgsnd(msgid,&msg_sbuf,sizeof(msg_sbuf.mtext),sflags);
	if(reval==-1)
	{
		printf("message send error\n");
	}
	//����һ����Ϣ�������Ϣ��������
	msg_stat(msgid,msg_ginfo);

	 
	reval=msgctl(msgid,IPC_RMID,NULL);//ɾ����Ϣ����
	if(reval==-1)
	{
		printf("unlink msg queue error\n");
	}
}
void msg_stat(int msgid,struct msqid_ds msg_info)
{
	int reval;
	sleep(1);//ֻ��Ϊ�˺������ʱ��ķ���
	reval=msgctl(msgid,IPC_STAT,&msg_info);
	if(reval==-1)
	{
		printf("get msg info error\n");
	}
	printf("\n");
	printf("current number of bytes on queue is %d\n",msg_info.msg_cbytes);
	printf("number of messages in queue is %d\n",msg_info.msg_qnum);
	printf("max number of bytes on queue is %d\n",msg_info.msg_qbytes);
	//ÿ����Ϣ���е��������ֽ�������������MSGMNB��ֵ�Ĵ�С��ϵͳ���졣�ڴ����µ���Ϣ����ʱ��//msg_qbytes��ȱʡֵ����MSGMNB
	printf("pid of last msgsnd is %d\n",msg_info.msg_lspid);
	printf("pid of last msgrcv is %d\n",msg_info.msg_lrpid);
	printf("last msgsnd time is %s", ctime(&(msg_info.msg_stime)));
	printf("last msgrcv time is %s", ctime(&(msg_info.msg_rtime)));
	printf("last change time is %s", ctime(&(msg_info.msg_ctime)));
	printf("msg uid is %d\n",msg_info.msg_perm.uid);
	printf("msg gid is %d\n",msg_info.msg_perm.gid);
}

```








