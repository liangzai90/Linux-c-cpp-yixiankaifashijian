



- > 文件类型

-: 普通文件

d: 目录

b: 块设备文件（例如硬盘，光驱等）

c: 字符设备文件（例如“猫”等串口设备 ）

l: 链接文件

p: 管道文件

s: 套接口文件/数据接口文件（例如启动一个MySQL服务器时会产生一个mysql.sock文件）


- > 打开一个文件
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


- > 循环读数据，每次读10个字节
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
    char filename[] = "/root/test.txt"; //要读取的文件

    fd = open(filename,O_RDONLY); //只读方式打开文件
    if(-1==fd)
    {   
            printf("Open file %s failuer,fd:%d\n",filename,fd);
		       return -1;
    }
    else  printf("Open file %s success,fd:%d\n",filename,fd);
  
    //循环读取数据，直到文件末尾或者出错
    while(size)
    {   
       //读取文件中的数据，10的意思是希望读10个字节，但真正读到的字节数是函数返回值
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
                for(i =0;i<size;i++) //循环打印文件读到的内容
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


- > 向文件写入数据
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

- > 获取文件状态
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


- > 建立文件和内存映射

  把文件映射到内存并打印出来

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
	write(fd, "\0", 1); /* 在文件最后添加一个空字符，以便下面printf正常工作 */  
	lseek(fd, 0, SEEK_SET);  
	mapped_mem =(char*) mmap(start_addr,
		flength,
		PROT_READ,        //允许读  
		MAP_PRIVATE,       //不允许其它进程访问此内存区域  
		fd,
		0);  
      
/* 使用映射区域. */  
	printf("%s\n", mapped_mem); /* 为了保证这里工作正常，参数传递的文件名最好是一个文本文件 */  
	close(fd);  
	munmap(mapped_mem, flength);  
	return 0;  
}  
```

  修改文件的内存映像
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
	write(fd, "\0", 1); //在文件最后添加一个空字符，以便下面printf正常工作 
	lseek(fd, 0, SEEK_SET);  
	start_addr = (void*)0x80000;  
	mapped_mem = (char*)mmap(start_addr,
		flength,
		PROT_READ|PROT_WRITE,        //允许写入  
		MAP_SHARED,       //允许其它进程访问此内存区域  
		fd,
		0);  
      
    // 使用映射区域. 
	printf("%s\n", mapped_mem); //为了保证这里工作正常，参数传递的文件名最好是一个文本文  
	while ((p = strstr(mapped_mem, "hello"))) { // 此处来修改文件 内容 
		memcpy(p, "Linux", 5);  
		p += 5;  
	}  
          
	close(fd);  
	munmap(mapped_mem, flength);  
	return 0;  
}  

```

- > 用C++流的方式读写文件
```cpp
#include <fstream>
#include <iostream>
using namespace std;
 
int main()
{
    
	char data[100];

	   // 以写模式打开文件
	ofstream outfile;
	outfile.open("afile.dat");

	cout << "Writing to the file" << endl;
	cout << "Enter your name: "; 
	cin.getline(data, 100);

	   // 向文件写入用户输入的数据
	outfile << data << endl;

	cout << "Enter your age: "; 
	cin >> data;
	cin.ignore();
   
	// 再次向文件写入用户输入的数据
	outfile << data << endl;

	   // 关闭打开的文件
	outfile.close();

	   // 以读模式打开文件
	ifstream infile; 
	infile.open("afile.dat"); 
 
	cout << "Reading from the file" << endl; 
	infile >> data; 

	   // 在屏幕上写入数据
	cout << data << endl;
   
	// 再次从文件读取数据，并显示它
	infile >> data; 
	cout << data << endl; 

	   // 关闭打开的文件
	infile.close();

	return 0;
}
```










