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


























