作为一名C++程序员，掌握好多线程并发开发技术是学习的重中之重。
而且为了能在实践中承接老代码系统的维护，学习C++11之前的多线程开发技术也是必不可少的，
而且以后开发新功能，C++11将是大势所趋。

具体到Linux C++开发环境，它提供了一套 POSIX API 函数来管理线程，
用户既可以直接使用这些POSIX API函数，也可以使用C++自带的线程类。

作为一名Linux C++开发者，这两者都应该会使用，因为在Linux C++程序中，
这两种方式都可能会出现。


### 线程的状态
一个线程从创建到结束是一个生命周期，总是处于下面4个状态中的一个。

 - > 就绪态：
    线程能够运行的条件已满足，只是在等待处理器。

 - > 运行态：
    运行态表示线程正在处理器中运行，正占用着处理器。

 - > 阻塞态：
     由于在等待处理器之外的其他条件而无法运行的状态叫做阻塞态。

 - > 终止态：
    终止态就是线程的线程函数运行结束或被其他线程取消后处于的状态。


### 线程函数 
线程函数就是线程创建后进入运行态后要执行的函数。
执行线程，说到底就是执行线程函数。这个函数是我们自定义的，
然后在创建线程时把我们的函数作为参数传入线程创建函数。

线程的函数可以是一个全局函数或类的静态函数，比如在POSIX线程库中，
它通常这样声明：
```cpp
void *ThreadProc(void *arg);
```

### 线程标识
通常线程创建成功之后会返回一个线程ID。

### 利用 POSIX 多线程 API 函数进行多线程开发
```txt
pthread_create            创建线程
---------------------------------------------------------------
pthread_join              等待一个线程的结束
---------------------------------------------------------------
pthread_self              获取线程ID
---------------------------------------------------------------
pthread_cancel            取消另一个线程
---------------------------------------------------------------
pthread_exit              在线程函数中调用来退出线程函数
---------------------------------------------------------------
pthread_kill              向线程发送一个信号
---------------------------------------------------------------
```
这些 API 函数需要包含头文件 pthread.h 并且在编译的时候要加上库 pthread，表示包含多线程库文件。


```cpp
int pthread_create(pthread_t *pid, const pthread_attr_t *attr, void *(*start_routine)(void *), void *arg);
```
 - > pid 是一个指针，指向创建成功后的线程的ID，pthread_t其实就是 unsigned long int;
 - > attr 是指向线程属性结构 pthread_attr_t 的指针，如果为 NULL ，则使用默认属性;
 - > start_routine 指向线程函数的地址，线程函数就是线程创建后要执行的函数;
 - > arg 指向传给线程函数的参数，如果执行成功，函数返回0


POSIX提供了函数 pthread_join 来等待子线程结束，
即子线程的线程函数执行完毕后，prthread_join 才返回，
因此 pthread_join 是一个阻塞函数。
函数 pthread_join 会让主线程挂起（休眠，就是让出CPU），直到子线程都退出，
同时 pthread_join 能让子线程所占资源得到释放。
子线程退出后，主线程会接收到系统的信号，从休眠中恢复。

```cpp
int pthread_join(pthread_t pid, void **value_ptr);
```
 - > pid 是所等待线程的ID号
 - > value_ptr 通常可以设置为NULL。
如果不为NULL，则 pthread_join 复制一份线程退出值到一个内存区域，
并让 *value_ptr 指向该内存区域，因此 pthread_join 还有一个重要的功能
就是能获得子线程的返回值。如果函数执行成功返回0，否则返回错误码。
 


### 创建一个简单的线程，不传参数
```cpp
// 要加上 pthread 库，2中表达方式
// g++ demo.cpp -pthread 
// g++ demo.cpp -lpthread 
//
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>  //sleep

void *thfunc(void *arg) //线程函数
{
    printf("in thfunc \r\n");
    return (void *)0;
}

int main(int argc, char *argv[])
{
    pthread_t tidp;
    int ret;

    ret = pthread_create(&tidp, NULL, thfunc, NULL); //创建线程
    if(ret)
    {
        printf("pthread_create failed:%d \r\n", ret);
        return -1;
    }

    sleep(1); //main线程挂起1秒钟，为了让子线程有机会执行
    printf("in main:thread is created\r\n");
    return 0;
}
```


### 创建一个线程，并传入整形参数 
```cpp
#include <pthread.h>
#include <stdio.h>

void *thfunc(void *arg)
{
    int *pn = (int *)(arg); //获取参数的地址
    int n = *pn;
    printf("in thfunc:n = %d \r\n", n);
    return (void *)0;
}

int main(int argc,char *argv[])
{
    pthread_t tidp;
    int ret, n = 110;

    ret = pthread_create(&tidp, NULL, thfunc, &n); // 创建线程并传递n的地址
    if(ret)
    {
        printf("pthread_create failed:%d \r\n", ret);
        return -1;
    }

    printf("tidp is :%d",tidp);

    // tipd 是线程的id 
    pthread_join(tidp, NULL); // 等待线程结束
    printf("in main:thread is created.\r\n");

    return 0;
}
```


### 创建一个线程，并传递字符串作为参数
```cpp
#include <pthread.h>
#include <stdio.h>

void *thfunc(void *arg)
{
    char *str;
    str = (char *)arg; //得到传递进来的字符串
    printf("in thfunc: str = %s \r\n", str);  // 打印字符串
    return (void *)0;
}

int main(int argc, char *argv[])
{
    pthread_t tidp;
    int ret;
    const char *str = "hello world";

    ret = pthread_create(&tidp, NULL, thfunc, (void *)str); // 创建线程并传递 str  
    if(ret)
    {
        printf("pthread_create failed:%d \r\n", ret);
        return -1;
    }
    pthread_join(tidp, NULL); // 等待子线程结束
    printf("in main:thread is created.\r\n");

    return 0;
}
```


### 创建一个线程，并传递结构体作为参数
```cpp
#include <pthread.h>
#include <stdio.h>

typedef struct   // 定义结构体的类型
{
    int n;
    char* str;
}MYSTRUCT;

void *thfunc(void *arg)
{
    MYSTRUCT *p = (MYSTRUCT*)arg;
    printf("in thfunc:n=%d, str=%s \r\n", p->n, p->str); //打印结构体内容
    return (void *)0;
}

int main(int argc, char *argv[])
{
    pthread_t tidp;
    int ret;
    MYSTRUCT  mystruct;  // 定义结构体 
    // 初始化结构体
    mystruct.n = 110;
    mystruct.str = "Hello world.";
    
    ret = pthread_create(&tidp, NULL, thfunc, (void *)&mystruct); // 创建线程并传递 结构体地址  
    if(ret)
    {
        printf("pthread_create failed:%d \r\n", ret);
        return -1;
    }
    pthread_join(tidp, NULL); // 等待子线程结束
    printf("in main:thread is created.\r\n");

    return 0;
}
```


### 创建一个线程，共享进程数据
```cpp
#include <pthread.h>
#include <stdio.h>

int gn = 10;   // 定义一个全局变量，将会在主线程和子线程中用到

void *thfunc(void *arg)
{
    gn++;  //递增1
    printf("in thfunc:gn=%d, \r\n", gn);  //打印全局变量 gn 值
    return (void *)0;
}

int main(int argc, char *argv[])
{
    pthread_t tidp;
    int ret;
    ret = pthread_create(&tidp, NULL, thfunc, NULL);
    if(ret)
    {
        printf("pthread_create failed:%d \r\n", ret);
        return -1;
    }

    pthread_join(tidp, NULL);  //等待子线程结束
    gn++; // 子线程结束后， gn 再递增1
    printf("in main: gn= %d \r\n", gn);  // 再次打印全局变量 gn 值 

    return 0;
}
```


### 线程的属性

```cpp
int pthread_getattr_np(pthread_t thread, pthread_attr_t *attr)
```
 - > thread 是线程ID
 - > attr 返回线程属性结构体的内容

如果函数执行成功就返回0，否则返回错误码。
使用该函数需要定义宏 _GNU_SOURCE，而且要在 pthread.h 前定义，
具体如下：
```cpp
#define  _GUN_SOURCE
#include <pthread.h>
```
当函数 pthread_getarrt_np 获得的属性结构体变量不再需要的时候，
应该用函数 pthread_attr_destroy 进行销毁。 


如果要创建非默认属性的线程，可以在创建线程之前用
函数 pthread_attr_init 来初始化一个线程属性结构体，
再调用 相应 API 函数来设置相应的属性，接着把属性结构体的地址参数
作为数传入 pthread_create。
```cpp
int pthread_attr_init(pthread_attr_t *attr);
```
 - > attr 为指向线程属性结构体的指针。如果函数执行成功就返回0，否则返回一个错误码。

需要注意：使用 pthread_attr_init 初始化线程属性，之后(传入 pthread_create)需要
使用 pthread_attr_destroy 销毁，从而释放相关资源。

```cpp
int pthread_attr_destroy(pthread_attr_t *attr);
```
 - > attr 为指向线程属性结构体的指针。如果函数执行成功就返回0，否则返回一个错误码。

POSIX 下的线程要么是分离的，要么是非分离状态的（也称可连接, joinable)。
默认情况下创建的线程是可连接的，一个可结合的线程是可以被其他线程收回资源
和杀死（或称取消）的，并且它不会主动释放资源（比如栈空间），
必须等待其他线程来回收其资源。

因此我们要在主线程使用 pthread_join 函数，该函数是一个阻塞函数，
当它返回时，所等待的线程的资源也就被释放了。

再次强调，如果是可连接的线程，当线程函数自己返回结束时或调用 pthread_exit 结束时，
都不会释放线程所占用的堆栈和线程描述符，必须调用 pthread_join 且返回后，
这些资源才会被释放。


一个可连接的线程所占用的内存仅当有线程对其执行 pthread_join 后才会释放，
为了避免内存泄漏，可连接的线程在终止时，要么被设置为 DETACHED(可分离)，
要么使用 pthread_join 来回收资源。


可分离的线程，这种线程运行结束时，其资源将立即被系统回收。
将一个线程设置为可分离状态有两种方式。
一种是调用函数 pthread_detach，将线程转换为可分离线程。
另一种是在创建线程时就将它设置为可分离状态，
基本过程是首先 初始化一个线程属性的结构体变量（通过函数 pthread_attr_init），
然后将其设置为可分离状态（通过函数 pthread_attr_setdetachstate），
最后将该结构体变量的地址作为参数传入线程创建函数 pthread_create，
这样所创建出来的线程就直接处于可分离状态：

```cpp
int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
```
 - > attr 是要设置的属性结构体；
 - > detachstate 是要设置的分离状态值，可以取值 PTHREAD_CREATE_DETACHED 或
 PTHREAD_CREATE_JOINABLE。如果函数执行成功就返回0，否则返回非零错误码。


### 创建一个可分离线程

```cpp
#include <iostream>
#include <unistd.h>  //sleep
#include <pthread.h>

using namespace std;

void *thfunc(void *arg)
{
    cout <<("sub thread is running \r\n");
    return nullptr;
}

int main(int argc, char *argv[])
{
    pthread_t thread_id;
    pthread_attr_t thread_attr;
    struct sched_param thread_param;
    size_t stack_size;
    int res;

    res = pthread_attr_init(&thread_attr);
    if(res)
    {
        cout<<"pthread_attr_init failed:"<<res<<endl;
    }

    res = pthread_attr_setdetachstate(&thread_attr, PTHREAD_CREATE_DETACHED);//设置为分离状态
    if(res)
    {
        cout <<"pthread_attr_setdetachstate failed:"<<res<<endl;
    }

    res = pthread_create(&thread_id, &thread_attr, thfunc, nullptr);

    if(res)
    {
        cout <<"pthread_create failed:"<<res<<endl;
    }

    sleep(1);
    return 0;
}
```

### 创建一个可分离线程，且 main 线程先退出
```cpp
#include <iostream>
#include <unistd.h>  //sleep
#include <pthread.h>

using namespace std;

void *thfunc(void *arg)
{
    cout <<("sub thread is running \r\n");
    return nullptr;
}

int main(int argc, char *argv[])
{
    pthread_t thread_id;
    pthread_attr_t thread_attr;
    struct sched_param thread_param;
    size_t stack_size;
    int res;

    res = pthread_attr_init(&thread_attr); //初始化线程结构体
    if(res)
    {
        cout<<"pthread_attr_init failed:"<<res<<endl;
    }

    res = pthread_attr_setdetachstate(&thread_attr, PTHREAD_CREATE_DETACHED);//设置为分离状态
    if(res)
    {
        cout <<"pthread_attr_setdetachstate failed:"<<res<<endl;
    }

    res = pthread_create(&thread_id, &thread_attr, thfunc, nullptr);

    if(res)
    {
        cout <<"pthread_create failed:"<<res<<endl;
    }

    cout <<"main thread will exit \r\n"<<endl;

    pthread_exit(nullptr);//主线程退出，但进程不会此刻退出，下面的语句不会再执行
    cout <<"main thread has exited, this line will not run \r\n"<<endl;

    return 0;
}

```


函数 pthread_detach 把一个可连接线程转变为一个可分离线程
```cpp
int pthread_detach(pthread_t thread);
```
 - > thread 是要设置为分离状态的线程的ID。

如果函数执行成功返回0，否则返回错误码。


获取分离状态函数 pthread_attr_getdetachstate 
```cpp
int pthread_attr_getdetachstate(pthread_attr_t *attr, int *detachstate);
```
 - > attr 为属性结构体指针
 - > detachstate 返回分离状态

 如果函数执行成功返回0，否则返回错误码。


### 获取线程的分离状态属性
```cpp
#ifndef _GNU_SOURCE  
#define _GNU_SOURCE     /* To get pthread_getattr_np() declaration */  
#endif  
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  
#include <unistd.h>  
#include <errno.h>  
      
#define handle_error_en(en, msg) \  
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)  
      

      
static void * thread_start(void *arg)  
{  
	int i,s;  
	pthread_attr_t gattr;  
       
	s = pthread_getattr_np(pthread_self(), &gattr);  
	if (s != 0)  
		handle_error_en(s, "pthread_getattr_np");  
      
	printf("Thread's detachstate attributes:\n");  
 
	s = pthread_attr_getdetachstate(&gattr, &i);  
	if (s)  
		handle_error_en(s, "pthread_attr_getdetachstate");  
	printf("Detach state        = %s\n",
		(i == PTHREAD_CREATE_DETACHED) ? "PTHREAD_CREATE_DETACHED" :  
		(i == PTHREAD_CREATE_JOINABLE) ? "PTHREAD_CREATE_JOINABLE" :  
		"???");  

	 pthread_attr_destroy(&gattr);  
}  
      
int main(int argc, char *argv[])  
{  
	pthread_t thr;  
	int s;  
 
	s = pthread_create(&thr, NULL, &thread_start, NULL);  
	if (s != 0)  
	{
		handle_error_en(s, "pthread_create"); 
		return 0;
	}
	
	pthread_join(thr, NULL); //等待子线程结束
}  
```

### 把可连接线程转换为可分离线程
```cpp
#ifndef _GNU_SOURCE  
#define _GNU_SOURCE     /* To get pthread_getattr_np() declaration */  
#endif  
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  
#include <unistd.h>  
#include <errno.h>  
     
static void * thread_start(void *arg)  
{  
	int i,s;  
	pthread_attr_t gattr;  
 
	s = pthread_getattr_np(pthread_self(), &gattr);  
	if (s != 0)  
		printf("pthread_getattr_np failed\n");  
    
	s = pthread_attr_getdetachstate(&gattr, &i);  
	if (s)  
		printf(  "pthread_attr_getdetachstate failed");  
	printf("Detach state        = %s\n",
		(i == PTHREAD_CREATE_DETACHED) ? "PTHREAD_CREATE_DETACHED" :  
		(i == PTHREAD_CREATE_JOINABLE) ? "PTHREAD_CREATE_JOINABLE" :  
		"???");  

	pthread_detach(pthread_self()); //转换线程为可分离线程
	
	s = pthread_getattr_np(pthread_self(), &gattr);  
	if (s != 0)  
		printf("pthread_getattr_np failed\n");  
	s = pthread_attr_getdetachstate(&gattr, &i);  
	if (s)  
		printf(" pthread_attr_getdetachstate failed");  
	printf("after pthread_detach,\nDetach state        = %s\n",
		(i == PTHREAD_CREATE_DETACHED) ? "PTHREAD_CREATE_DETACHED" :  
		(i == PTHREAD_CREATE_JOINABLE) ? "PTHREAD_CREATE_JOINABLE" :  
		"???");  
	
	 pthread_attr_destroy(&gattr);  //销毁属性
}  
      
int main(int argc, char *argv[])  
{  
	pthread_t thread_id;  
	int s;  
 
	s = pthread_create(&thread_id, NULL, &thread_start, NULL);  
	if (s != 0)  
	{
		printf("pthread_create failed\n"); 
		return 0;
	}
	pthread_exit(NULL);//主线程退出，但进程并不马上结束
}  

```


### 栈尺寸
在线程函数开设局部变量（尤其是数组）不要超过默认栈尺寸大小。
获取线程栈尺寸属性的函数是 pthread_attr_getstacksize
```cpp
int pthread_attr_getstacksize(pthread_attr_t *attr, size_t *stacksize);
```
 - > attr 指向属性结构体
 - > stacksize 用于获得栈尺寸（单位是字节），指向 size_t 类型的变量。

 如果函数执行成功返回0，否则返回错误码。

```cpp
#ifndef _GNU_SOURCE  
#define _GNU_SOURCE     /* To get pthread_getattr_np() declaration */  
#endif  
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  
#include <unistd.h>  
#include <errno.h>  
#include <limits.h>
static void * thread_start(void *arg)  
{  
	int i,res;  
	size_t stack_size;
	pthread_attr_t gattr;  
 
	res = pthread_getattr_np(pthread_self(), &gattr);  
	if (res)  
		printf("pthread_getattr_np failed\n");  
    
	res = pthread_attr_getstacksize(&gattr, &stack_size);
	if (res)
		printf("pthread_getattr_np failed\n"); 
	
	printf("Default stack size is %u byte; minimum is %u byte\n", stack_size, PTHREAD_STACK_MIN);
	 
	
	 pthread_attr_destroy(&gattr);  
}  
      
int main(int argc, char *argv[])  
{  
	pthread_t thread_id;  
	int s;  
 
	s = pthread_create(&thread_id, NULL, &thread_start, NULL);  
	if (s != 0)  
	{
		printf("pthread_create failed\n"); 
		return 0;
	}
	pthread_join(thread_id, NULL); //等待子线程结束
}  
```














