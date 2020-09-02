
���̣�Process���ǲ���ϵͳ�ṹ�Ļ�����
������һ�����ж������ܵĳ����ĳ�����ݼ��ڴ�����ϵ�ִ�й��̣�
����Ҳ����Ϊ����Դ���䡿��һ��������λ��
Linux��Ϊ����û����������Ĳ���ϵͳ���ض�֧�ֶ���̡�
��������ִ�����ϵͳ�Ļ���������

- > ��ȡ��ǰ����ID (getpid())
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

- > ͨ��fork�������ӽ���
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
  ������ݣ�
```txt
I am the parent process, my pid is 2825
fpid =2826
count=1
I am the child process, my pid is  2826
count=1
```
fork���õ�һ������֮������������������һ�Σ�ȴ�ܹ��������Ρ�
fork�ǰѽ��̵�ǰ���������һ�ݣ�ִ��forkʱ�������Ѿ�ִ���������int count =0;��
forkֻ������һ��Ҫִ�еĴ��뵽�µĽ��̡�


- > ʹ��exec��������

#include <unistd.h>
int execl(const char* path, const char* arg, ...);

����pathָ��Ҫִ�е��ļ�·����
����Ĳ�����arg,...������ִ�иó���ʱ���ݵĲ����б�
���ҵ�1������Ϊ��argv[0](��path����Ĳ�������Ϊ��argv[0])����2������Ϊ��argv[1]....
���һ�����������ǿ�ָ�� NULL������
�����ɹ�ʱ������ֵ��ʧ���򷵻�-1��ʧ��ԭ����errno�У���ͨ��perror()��ӡ��

����Ҫע�⣬����ϵͳ������򣬱���pwd���argv[0]�Ǳ���Ҫ�еģ�����ֵ������һ����������ַ���

ʹ��execlִ�в���ѡ����������pwd
```cpp
#include <unistd.h>  

int main(int argc, char* argv[])
{
	execl("/bin/pwd", "asdfaf",  NULL);  
	
	return 0;
}
```

ʹ��execlִ�д�ѡ������� ����ls
```cpp
 
#include <unistd.h>  
      
int main(void)  
{  
    //ִ��/binĿ¼�µ�ls  
    //��һ������Ϊ������ls���ڶ�������Ϊ-al,����������Ϊ/etc/passwd  
	execl("/bin/ls", "ls", "-al", "/etc/passwd", NULL);  
	return 0;  
}  
```

ʹ��execlִ�����ǵĳ���
```cpp

/////�����ɵĶ������ļ�����Ϊ mytest
/////�����ɵĶ������ļ�����Ϊ mytest
/////�����ɵĶ������ļ�����Ϊ mytest

#include <string.h>
using namespace std;
#include <iostream>
 
int main(int argc, char* argv[])
{
	int i;
	cout <<"argc=" << argc << endl; //��ӡ�´������Ĳ�������
	
	for(i=0;i<argc;i++)   //��ӡ��������
		cout<<argv[i]<<endl;
	 
	if (argc == 2&&strcmp(argv[1], "-p")==0)  //�ж��Ƿ�������˲���-p
		cout << "will print all" << endl;
	else 
		cout << "will print little" << endl;
	
	cout << "my program over" << endl;
	return 0;
}



/////����ļ�ִ���������ɵĶ������ļ�
#include <unistd.h>
using namespace std;
#include <iostream>
 
int main(int argc, char* argv[])
{
	cout <<"111---------------------------------"<<endl;

	//��������
	//execl("./mytest", NULL);

	cout <<"222==============================="<<endl;

	//��2������
	execl("./mytest", "henry", "-p", NULL);

	//�ر�ע�⣬���execlִ�гɹ����������д����ǲ�������ģ�����
	cout <<"333==============================="<<endl;

	return 0;
}

```

- > ����execlp
execlp�������PATH����������ָ��Ŀ¼�в��ҷ��ϲ���file���ļ������ҵ����ִ�и��ļ���
Ȼ�󽫵ڶ����Ժ�Ĳ����������ļ���argv[0]��argv[1]...�����һ�����������ÿ�ָ��(NULL)������

execlp�����������£�

#include <unistd.h>
int execlp(const char* file, const char* arg, ...);

���ִ�гɹ������������أ�ִ��ʧ����ֱ�ӷ���-1��ʧ�ܵ�ԭ�����errno�С�


ʹ��execlpִ�в���ѡ����������pwd
execlp�ĵ�һ������ֱ����pwd���������򼴿ɣ�������Ҫд����ȫ·����
��Ϊ��������PATH���Ѿ�����·�� /usr/bin ��
��2�����������У������������ַ���.
```cpp
#include <unistd.h>  

int main(int argc, char* argv[])
{
	execlp("pwd", "",  NULL); 
	return 0;
}

```

- > ʹ��execlpִ�����ǵĳ���
```cpp

//�����ɵ��ļ�����ΪexeclpTest����Ϊ��2����ִ�г���Ҫִ��execlpTest)
//�����ɵ��ļ�����ΪexeclpTest����Ϊ��2����ִ�г���Ҫִ��execlpTest)
//�����ɵ��ļ�����ΪexeclpTest����Ϊ��2����ִ�г���Ҫִ��execlpTest)

#include <string.h>
using namespace std;
#include <iostream>
 
int main(int argc, char* argv[])
{
	cout <<"execlp test  ��ӡ����Ĳ����������͸�������"<<endl;
	int i;
	cout <<"argc=" << argc << endl; //��ӡ�´������Ĳ�������
	
	for(i=0;i<argc;i++)   //��ӡ��������
		cout<<argv[i]<<endl;
}



///// ���������õ������·����Ҳ���԰�execlpTest����ļ���·����ӵ�ϵͳĿ¼���棬
//Ȼ��execlp�ĵ�һ�������Ϳ���ֱ�Ӵ��ļ���
#include <unistd.h>  
using namespace std;
#include <iostream>

int main(int argc, char* argv[])
{
	//execlp("./execlpTest", NULL); //�����κβ�����mytest
	//cout << "111------------------\n";//���execlpִ�гɹ�����һ�䲻��ִ�е��ġ�

	execlp("./execlpTest", "henry", "he", NULL);
	cout << "222------------------\n";//���execlpִ�гɹ�����һ�䲻��ִ�е��ġ�
	
	return 0;
}

```


### ���̵���
���̵���Ҳ���Ǵ�������ȡ��ڶ��������ƻ����У��������������ڴ���������⽫���¶������
�Դ������Դ���໥���ᡣ���̵��ȵ������ǿ��ƺ�Э�����̶�CPU�ľ���������һ���ĵ����㷨
ʹĳһ��������ȡ��CPU�Ŀ���Ȩ���Ӷ�תΪ����״̬��

���̵��ȵĹ�����Ҫ��������¼ϵͳ�����н��̵�ִ��״̬������һ���ĵ����㷨���Ӿ�������
��ѡ��һ��������׼���Ѵ����������������������������̣������������л�����ѡ�н���
�Ľ��̿��ƿ����йص��ֳ���Ϣ�������״̬�֡�ͨ�üĴ��������ݣ����봦������Ӧ�ļĴ����У�
�Ӷ�����ռ�ô�������С�

���̵ĵ���һ���������������·�����
1.����ִ�еĽ����������
2.����ִ�еĽ��̵�������ԭ�ｫ�Լ���������������ȴ�״̬
3.ִ���еĽ������I/O���������
4.����ִ�еĽ��̵�����Pԭ�����������Դ�ò�������������������ߵ���Vԭ������ͷ�����Դ���Ӷ������˵ȴ���Ӧ��Դ�Ľ��̶���
5.�ڷ�ʱϵͳ�У�ʱ��Ƭ�Ѿ�����
6.���������е�ĳ�����̵����ȼ���ø��ڵ�ǰ���н��̵����ȼ����Ӷ�������̵ĵ���


## �����Ľ��̵����㷨
1.�����ȷ���FCFS��
2.ʱ��Ƭ��ת����RR��
3.���ȼ��㷨
4.�༶�������з�


## ���̵���
���в����ⷽ���֪ʶ

### �ػ�����
�ػ����̣�Daemon Process���������ں�̨��һ��������̡�
�������ڿ����ն˲��������Ե�ִ��ĳ�������ȴ�����ĳЩ���������顣
�ػ�������һ�ֺ����õĽ��̡�Linux�д�����������������ػ�����ʵ�ֵģ�
���� Internet ������ inetd��Web������ httpd �ȣ����⻹�г������ػ����̰���ϵͳ
��־���� syslogd�����ݿ������ mysld�ȡ�

�ػ����̵��ص㣺
1.�ػ����̶����г����û���Ȩ��
2.�ػ����̵ĸ�������init����
3.�ػ����̶����ÿ����նˣ���TTY���� "?"��ʾ��TPGIDΪ -1 
4.�ػ����̶��Ǹ��Խ�����ϻỰ���̵�Ψһ����


�鿴�ػ����̣�
```bash
ps axj
```


End