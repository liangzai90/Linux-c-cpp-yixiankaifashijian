
### 不用线程同步的多线程累加
```cpp
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>
#include <sys/time.h> 
#include <string.h>
#include <cstdlib>
 
int gcn = 0;
	
void *thread_1(void *arg) {
	int j;
	for (j = 0; j < 10000000; j++) {
	 
		gcn++;
		 
	}  
	pthread_exit((void *)0);
}

void *thread_2(void *arg) {
	int j;
	for (j = 0; j < 10000000; j++) {
	 
		gcn++;
		 
	}  
	pthread_exit((void *)0);
}
int main(void) 
{
	int j,err;
	pthread_t th1, th2;
	 
	for (j = 0; j < 10; j++)
	{
		err = pthread_create(&th1, NULL, thread_1, (void *)0);
		if (err != 0) {
			printf("create new thread error:%s\n", strerror(err));
			exit(0);
		}  
		err = pthread_create(&th2, NULL, thread_2, (void *)0);
		if (err != 0) {
			printf("create new thread error:%s\n", strerror(err));
			exit(0);
		}  
           
		err = pthread_join(th1, NULL);
		if (err != 0) {
			printf("wait thread done error:%s\n", strerror(err));
			exit(1);
		}
		err = pthread_join(th2, NULL);
		if (err != 0) {
			printf("wait thread done error:%s\n", strerror(err));
			exit(1);
		}
		printf("gcn=%d\n", gcn);
		gcn = 0;
	}
	 

	return 0;
}
```

### 不用线程同步的卖货程序
```cpp
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>
 
int a = 200;
int b = 100;
 
void* ThreadA(void*)
{
	while (1)
	{
		a -= 50; //卖出价值50元的货物 
		b += 50;//收回50元钱
	}
}
 
void* ThreadB(void*)
{
	while (1)
	{
		printf("%d\n", a + b);
		sleep(1);    
	}
}
 
int main()
{
	pthread_t tida, tidb;
 
	pthread_create(&tida, NULL, ThreadA, NULL);
	pthread_create(&tidb, NULL, ThreadB, NULL);
	pthread_join(tida, NULL);
	pthread_join(tidb, NULL);
	return 1;
}
```


### 利用 POSIX 多线程 API 函数进行线程同步

POSIX 提供了3种方式进行线程同步，即互斥锁、读写锁和条件变量。

互斥锁（也可称互斥量）是线程同步的一种机制，用来保护多线程的共享资源。
同一时刻，只允许一个线程对临界区进行访问。

互斥锁的工作流程是：
初始化一个互斥锁，在进入临界区前把互斥锁加锁
（防止其他线程进入临界区），退出临界区的时候把互斥锁解锁
（让别的线程有机会进入临界区），
最后不用互斥锁的时候就销毁它。

POSIX 库中用类型 pthread_mutex_t 来定义一个互斥锁。

pthread_mutex_t 是一个联合体类型，定义在 pthreadtypes.h中。
使用的时候，包含头文件 <pthread.h>即可
我们可以如下定义一个互斥变量：
pthread_mutex_t  mutex;

### 互斥锁的初始化
【动态初始化/函数初始化】 互斥锁：

函数是：pthread_mutex_init 

声明如下：
```cpp
int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);
```
 - > mutex 是指向 pthread_mutex_t变量的指针
 - > attr 是指向 pthread_mutexattr_t 的指针，表示互斥锁的属性，
 如果赋值NULL，则使用默认的互斥锁属性。

 如果函数执行成功返回0，否则返回错误码。
注意 关键字 restrict 只用于限定指针。


【静态初始化】 互斥锁：

用宏 PTHREAD_MUTEX_INITIALIZER 来静态地初始化互斥锁（这种
方式叫常量初始化）
```cpp
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
```
注意，如果mutex是指针，不能用这种静态方式赋值，只能通过函数赋值。

注意，静态初始化的互斥锁是不需要销毁的，而动态初始化的互斥锁是需要销毁的。

### 互斥锁的上锁和解锁
用于上锁的函数是 pthread_mutex_lock 或 pthread_mutex_trylock。
```cpp
int pthread_mutex_lock(pthread_mutex_t *mutex);
```
mutex 是指向 pthread_mutex_t 变量的指针，应该已经成功初始化过。
函数执行成功返回0，否则返回错误码。

值得注意的是，如果调用该函数时互斥锁已经被其他线程上锁了，
则调用该函数的线程将阻塞。

另一个上锁函数 pthread_mutex_trylock 在调用时，
如果互斥锁已经上锁了，则并不阻塞，而是立即返回，
并且函数返回 EBUSY
```cpp
int pthread_mutex_trylock(pthread_mutex_t *mutex);
```
mutex是指向 pthread_mutex_t 变量的指针，应该已经成功初始化过。
函数执行成功返回0，否则返回错误码。

对临界区解锁，解锁函数是 pthread_mutex_unlock 
```cpp
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
mutex是指向pthread_mutex_t变量的指针，应该是已经上锁的互斥锁。
函数执行成功返回0，否则返回错误码。
pthread_mutex_unlock 要和 pthread_mutex_lock成对使用。


互斥锁的销毁，函数是 pthread_mutex_destroy
```cpp
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```
mutex是指向pthread_mutex_t变量的指针，应该是已经初始化的互斥锁。
函数执行成功时返回0，否则返回错误码。


### 使用互斥锁的多线程累加
```cpp
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>
#include <sys/time.h>
#include <string.h>
#include <cstdlib>
#include <iostream>
using namespace std;
int gcn = 0;

pthread_mutex_t  mutex;

void *thread_1(void *arg){
	for(int j=0;j<1000000;j++)
	{
		pthread_mutex_lock(&mutex);
		gcn++;
		pthread_mutex_unlock(&mutex);
	}
	pthread_exit((void *)0);
}


void *thread_2(void *arg){
	for(int j=0;j<1000000;j++)
	{
		pthread_mutex_lock(&mutex);
		gcn++;
		pthread_mutex_unlock(&mutex);
	}
	pthread_exit((void *)0);
}

int main(void)
{
	pthread_t  th1, th2;
	pthread_mutex_init(&mutex, NULL);//初始化互斥锁
	for(int j=0;j<10;++j)
	{
		int err = pthread_create(&th1, NULL, thread_1, (void *)0);
		if(err!=0)
		{
			printf("create new thread A error:%s\r\n", strerror(err));
			exit(0);
		}

		err = pthread_create(&th2, NULL, thread_2, (void *)0);
		if(err !=0)
		{
			printf("create new thread B error:%s\r\n", strerror(err));
			exit(0);
		}

		err = pthread_join(th1,NULL);
		if(err != 0)
		{
			printf("wait thread A done ERROR:%s\r\n", strerror(err));
			exit(1);
		}

		err = pthread_join(th2,NULL);
		if(err != 0)
		{
			printf("wait thread B done ERROR:%s\r\n", strerror(err));
			exit(1);
		}

		printf("gcn=%d \r\n",gcn);
		gcn = 0;
	}

	pthread_mutex_destroy(&mutex);//销毁互斥锁
	cout <<endl<<"_______GAME OVER_______"<<endl;
	return 0;
}
```


### 使用互斥锁进行同步的卖货程序
```cpp
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>
#include <sys/time.h>
#include <string.h>
#include <cstdlib>
#include <iostream>
using namespace std;

int a = 200; 
int b = 100;

pthread_mutex_t  lock;//定义一个全局的互斥锁

void* ThreadA(void*)
{
	while(1){
		pthread_mutex_lock(&lock); //上锁
		a -= 50; //卖出50元的货物
		b += 50; //收回50元的货物
		pthread_mutex_unlock(&lock); //解锁
	}
}

void* ThreadB(void*)
{
	while(1){
		pthread_mutex_lock(&lock); //上锁
		printf("%d \r\n",a+b);
		pthread_mutex_unlock(&lock); //解锁
		sleep(1);
	}
}

int main(void)
{
	pthread_t  tida, tidb;
	pthread_mutex_init(&lock, NULL); //初始化互斥锁

	pthread_create(&tida, NULL, ThreadA, NULL);//创建伙计卖货线程

	pthread_create(&tidb, NULL, ThreadB, NULL);//创建老板对账线程

	pthread_join(tida, NULL);
	pthread_join(tidb,NULL);

	pthread_mutex_destroy(&lock);//销毁互斥锁

	return 0;
}
```

## 读写锁














































