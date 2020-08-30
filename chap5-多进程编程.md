
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












































