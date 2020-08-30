



- > �ļ�����

-: ��ͨ�ļ�

d: Ŀ¼

b: ���豸�ļ�������Ӳ�̣������ȣ�

c: �ַ��豸�ļ������硰è���ȴ����豸 ��

l: �����ļ�

p: �ܵ��ļ�

s: �׽ӿ��ļ�/���ݽӿ��ļ�����������һ��MySQL������ʱ�����һ��mysql.sock�ļ���


- > ��һ���ļ�
```cpp
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>

int main(void)
{
    int fd = -1;
    char filename[] = "test.txt";
    fd = open(filename,O_CREAT | O_RDWR, S_IRWXU);
    if(fd == -1)
    {
        printf("fail to open file %s, fd:%d \r\n", filename, fd);
    }
    else
    {
        printf("Open file %s successfully, fd:%d \r\n", filename, fd);
    }

    close(fd);
    return 0;    
}

```


- > ѭ�������ݣ�ÿ�ζ�10���ֽ�
```cpp
#include<stdio.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

int main(void)
{
    int fd = -1,i;
    ssize_t size =-1;
    char buf[10];   
    char filename[] = "/root/test.txt"; //Ҫ��ȡ���ļ�

    fd = open(filename,O_RDONLY); //ֻ����ʽ���ļ�
    if(-1==fd)
    {   
            printf("Open file %s failuer,fd:%d\n",filename,fd);
		       return -1;
    }
    else  printf("Open file %s success,fd:%d\n",filename,fd);
  
    //ѭ����ȡ���ݣ�ֱ���ļ�ĩβ���߳���
    while(size)
    {   
       //��ȡ�ļ��е����ݣ�10����˼��ϣ����10���ֽڣ��������������ֽ����Ǻ�������ֵ
        size = read(fd,buf,10);       
        if(-1==size)
        {   
            close(fd);
            printf("Read file %s error occurs\n",filename);
            return -1; 
        }else{
            if(size>0)
            {   
                printf("read %d bytes:",size);
                printf("\"");
                for(i =0;i<size;i++) //ѭ����ӡ�ļ�����������
                    printf("%c",*(buf+i));
                printf("\"\n");
            }else{
                printf("reach the end of file \n");
            }

        }
    }
    return 0;
}
```


- > ���ļ�д������
```cpp
#include<sys/types.h>
#include<sys/stat.h>
#include<stdio.h>
#include<unistd.h>
#include<fcntl.h>

int main(void)
{
    int fd = -1,i;
    ssize_t size =-1;
    int input =0; 
    
    char buf[] = "boys and girls\n hi,children!";  
    char filename[] = "test.txt";
    
    fd = open(filename,O_RDWR|O_APPEND);  
    if(-1==fd)
    {   
        printf("Open file %s faliluer \n",filename );
    }else{
        printf("Open file %s success \n,=",filename );
    }   
    size = write(fd,buf,strlen(buf));  
    printf("write %d bytes to file %s\n",size,filename);
    
    close(fd);
    return 0;
}
```

- > ��ȡ�ļ�״̬
```cpp
#include<sys/stat.h>
#include<sys/types.h>
#include<unistd.h>
#include <iostream>
using namespace std;

int main(void)
{
	struct stat st; 
    
	if (-1 == stat("/root/test.txt", &st))  
	{   
		cout << ("stat failed\n");
		return -1; 
	}   
	cout<<"file length:"<<st.st_size<<"byte"<<endl; 
	cout << "mod time:" << st.st_mtime << endl;  
	cout << "node:" << st.st_ino << endl;  
	cout << "mode:" << st.st_mode << endl;  
}
```


- > �����ļ����ڴ�ӳ��

  ���ļ�ӳ�䵽�ڴ沢��ӡ����

```cpp
#include <sys/mman.h> /* for mmap and munmap */  
#include <sys/types.h> /* for open */  
#include <sys/stat.h> /* for open */  
#include <fcntl.h>     /* for open */  
#include <unistd.h>    /* for lseek and write */  
#include <stdio.h>  
  
int main(int argc, char **argv)  
{  
	int fd;  
	char *mapped_mem, * p;  
	int flength = 1024;  
	void * start_addr = 0;  
  
	fd = open(argv[1], O_RDWR | O_CREAT, S_IRUSR | S_IWUSR);  
	flength = lseek(fd, 1, SEEK_END);  
	write(fd, "\0", 1); /* ���ļ�������һ�����ַ����Ա�����printf�������� */  
	lseek(fd, 0, SEEK_SET);  
	mapped_mem =(char*) mmap(start_addr,
		flength,
		PROT_READ,        //�����  
		MAP_PRIVATE,       //�������������̷��ʴ��ڴ�����  
		fd,
		0);  
      
/* ʹ��ӳ������. */  
	printf("%s\n", mapped_mem); /* Ϊ�˱�֤���﹤���������������ݵ��ļ��������һ���ı��ļ� */  
	close(fd);  
	munmap(mapped_mem, flength);  
	return 0;  
}  
```

  �޸��ļ����ڴ�ӳ��
```cpp
#include <sys/mman.h> /* for mmap and munmap */  
#include <sys/types.h> /* for open */  
#include <sys/stat.h> /* for open */  
#include <fcntl.h>     /* for open */  
#include <unistd.h>    /* for lseek and write */  
#include <stdio.h>  
#include <string.h> /* for memcpy */  
      
int main(int argc, char **argv)  
{  
	int fd;  
	char *mapped_mem, * p;  
	int flength = 1024;  
	void * start_addr = 0;  
      
	fd = open(argv[1], O_RDWR | O_CREAT, S_IRUSR | S_IWUSR);  
	flength = lseek(fd, 1, SEEK_END);  
	write(fd, "\0", 1); //���ļ�������һ�����ַ����Ա�����printf�������� 
	lseek(fd, 0, SEEK_SET);  
	start_addr = (void*)0x80000;  
	mapped_mem = (char*)mmap(start_addr,
		flength,
		PROT_READ|PROT_WRITE,        //����д��  
		MAP_SHARED,       //�����������̷��ʴ��ڴ�����  
		fd,
		0);  
      
    // ʹ��ӳ������. 
	printf("%s\n", mapped_mem); //Ϊ�˱�֤���﹤���������������ݵ��ļ��������һ���ı���  
	while ((p = strstr(mapped_mem, "hello"))) { // �˴����޸��ļ� ���� 
		memcpy(p, "Linux", 5);  
		p += 5;  
	}  
          
	close(fd);  
	munmap(mapped_mem, flength);  
	return 0;  
}  

```

- > ��C++���ķ�ʽ��д�ļ�
```cpp
#include <fstream>
#include <iostream>
using namespace std;
 
int main()
{
    
	char data[100];

	   // ��дģʽ���ļ�
	ofstream outfile;
	outfile.open("afile.dat");

	cout << "Writing to the file" << endl;
	cout << "Enter your name: "; 
	cin.getline(data, 100);

	   // ���ļ�д���û����������
	outfile << data << endl;

	cout << "Enter your age: "; 
	cin >> data;
	cin.ignore();
   
	// �ٴ����ļ�д���û����������
	outfile << data << endl;

	   // �رմ򿪵��ļ�
	outfile.close();

	   // �Զ�ģʽ���ļ�
	ifstream infile; 
	infile.open("afile.dat"); 
 
	cout << "Reading from the file" << endl; 
	infile >> data; 

	   // ����Ļ��д������
	cout << data << endl;
   
	// �ٴδ��ļ���ȡ���ݣ�����ʾ��
	infile >> data; 
	cout << data << endl; 

	   // �رմ򿪵��ļ�
	infile.close();

	return 0;
}
```










