
Linux�еĽ���Ϊ������ͬһ��������Э�����������Ǳ˴�֮������ܹ�����ͨ�š�
����һ������ϵͳ��˵�����̼��ͨ���ǲ��ɻ�ȱ�ġ�
Linux֧�ֶ��ֲ�ͬ��ʽ�Ľ��̼�ͨ�Ż��ƣ����źš��ܵ���FIFO��System V IPC���ƣ�
����System V IPC���ư������ź�������Ϣ���к͹����ڴ�3�ֻ��ơ�
Linux�µ���Щ���̼�ͨ�Ż��ƻ������Ǵ�UNIXƽ̨�ϵĽ��̼�ͨ�Ż��Ƽ̳кͷ�չ�����ġ�


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



















































