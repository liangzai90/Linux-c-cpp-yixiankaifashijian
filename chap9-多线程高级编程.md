
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
读写锁是多线程同步的另一种机制。

1.如果一个线程用读锁锁定了临界区，
那么其他线程也可以用读锁来进入临界区，
这样就可以有多个线程并行操作。
这个时候，如果再用写锁加锁就会发生阻塞，
写锁请求阻塞后，后面继续有读锁来请求时，
这些后来的读锁都将会被阻塞。
这样避免了读锁长期占用资源，防止写锁饥饿。

2.如果一个线程用写锁锁住了临界区，
那么其他线程无论是读锁还是写锁都会发生阻塞。

POSIX 库中用类型 pthread_rwlock_t 来定义一个互斥锁，
pthread_rwlock_t 是一个联合体类型。


### 读写锁的初始化
```cpp
pthread_rwlock_t  rwlock = PTHREAD_RWLOCK_INITIALIZER;


int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, 
const pthread_rwlockattr_t *restrict attr);
```


### 读写锁的上锁和解锁
读模式下的上锁函数有 pthread_rwlock_rdlock 和
pthread_rwlock_tryrdlock。

```cpp
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);

int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);

```

写模式下的读写锁： pthread_rwlock_wrlock 和 
pthread_rwlock_trywrlock 。
```cpp
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);

int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);

```

解锁
```cpp
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);

```

读写锁的销毁
```cpp
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
```

### 互斥锁和读写锁速度大PK
```cpp
```

条件变量 pthread_cond_t
```cpp
#include <pthread.h>
pthread_cond_t  cond;
```

条件变量初始化
```cpp
静态初始化
pthread_cond_t cond =  PTHREAD_COND_INITIALIZER;


动态初始化
int pthread_cond_init(pthread_cond_t* cond, pthread_condattr_t* cond_attr);

```

等待条件变量
```cpp
int pthread_cond_wait(pthread_cond_t* restrict cond,
pthread_mutex_t * restrict mutex);

```
为了防止多个线程同时请求函数 pthread_cond_wait 形成竞争，
因此条件变量必须和一个互斥锁联合使用。

如果条件不满足，调用 pthread_cond_wait 会发生这些
原子操作：
线程将 mutex 解锁、线程被条件变量cond阻塞。
这是一个原子操作，不会被打断。


唤醒等待条件变量的线程
```cpp
// 唤醒一个等待该条件变量的线程
int pthread_cond_signal(pthread_cond_t* cond);

// 唤醒所有等待该条件变量的线程
int pthread_cond_broadcase(pthread_cond_t* cond);
```

条件变量的销毁
```cpp
int pthread_cond_destroy(pthread_cond_t *cond);
```

### 找出1~20中能整除3的整数
```cpp
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER; //初始化互斥锁
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;//初始化条件变量

void *thread1(void *);
void *thread2(void *);

int i=1;  //注意，这里的i是全局变量

int main(void)
{
	pthread_t t_a;
	pthread_t t_b;

	pthread_create(&t_a, NULL, thread2, (void *)NULL);//创建线程 t_a
	pthread_create(&t_b, NULL, thread1, (void *)NULL);//创建线程 t_b

	pthread_join(t_b,NULL);//等待进程 t_b 结束

	pthread_mutex_destroy(&mutex);
	pthread_cond_destroy(&cond);
	
	exit(0);
	return 0;
}

void *thread1(void *junk)
{
	  //注意，这里的i是全局变量
	for(i=1;i<=20;i++)
	{
		pthread_mutex_lock(&mutex);//锁住互斥锁

		if(i%3 ==0)
		{
			pthread_cond_signal(&cond);//唤醒等待条件变量 cond 的线程
		}
		else 
		{
			printf("thread1:%d\r\n", i);//打印不能整除3 的i
		}
		pthread_mutex_unlock(&mutex);//解锁互斥锁
		sleep(1);
	}
}



void *thread2(void *junk)
{
	while(i<20){
		printf("-------->>>----while:%d\r\n",i);
		pthread_mutex_lock(&mutex);//锁住互斥锁

		if(i%3 !=0)
		{
			// pthread_cond_wait 会对mutex解锁
			pthread_cond_wait(&cond, &mutex);//等待条件变量
		}

		printf("---------------------thread2:%d\r\n",i);//打印能整除3的i

		pthread_mutex_unlock(&mutex);//解锁互斥锁
		sleep(1);
		i++;
	}
}
```


## C++11/14 中的线程同步
C++11/14 提供了两种方式进行线程同步，
即互斥锁和条件变量。
一线实际编程中用的较多的是互斥锁。

互斥锁的工作流程是：
初始化一个互斥锁，
在进入临界区前把互斥锁加锁（防止其他线程进入临界区），
退出临界区的时候把互斥锁解锁（让别的线程有机会进入临界区），
最后不用互斥锁的时候就销毁它。

C++11中与互斥锁相关的类（包括锁类型）
和函数都声明在头文件<mutex>中，
如果需要使用互斥锁相关的类，
就必须包含头文件<mutex>。

C++11 中的互斥锁有4种，并对应着4种不同的类。

 - > 基本互斥锁，对应的类为 std::mutex
 - > 递归互斥锁，对应的类为 std::recursive_mutex
 - > 定时互斥锁，对应的类为 std::time_mutex
 - > 定时递归互斥锁，对应的类为 std::time_mutex


### 基本互斥锁 std::mutex 
类 std::mutex 是最基本的互斥锁，用来同步线程对临界资源的互斥访问。
```txt
         类 std::mutex 的成员函数
-------------------------------------------------
mutex           构造函数
-------------------------------------------------
lock            互斥锁上锁
-------------------------------------------------
Try_lock        如果互斥锁没有上锁，则上锁
-------------------------------------------------
native_handle   得到本地互斥锁句柄
-------------------------------------------------
```


### 多线程统计计数器到10万
```cpp
#include <iostream>  // std::cout
#include <thread>    // std::thread
#include <mutex>     // std::mutex
using namespace std;

volatile int counter(0);  //定义一个全局变量，当作计数器，用于累加
std::mutex mtx; //用于包含 counter 的互斥锁

void thrfunc()
{
	for(int i=0;i<10000;++i)
	{
		mtx.lock(); // 互斥锁上锁
		++counter;  // 计数器累加
		mtx.unlock(); // 互斥锁解锁 
	}
}

int main(int argc, const char* argv[])
{
	std::thread threads[10];

	for(int i=0;i<10;++i)
	{
		threads[i] = std::thread(thrfunc); // 启动10个线程
	}

	for(auto & th:threads)
	{
		th.join();//等待10个线程结束
	}

	cout <<"count to "<<counter<<" successfully "<<endl;

	return 0;
}
```


### 定时互斥锁 std::time_mutex 
类 std::time_mutex 是定时互斥锁类，和基本互斥锁类似，
用来同步线程对临界资源的互斥访问，
区别是多了定时。
```cpp
        类 std::time_mutex 的成员函数
----------------------------------------------------------------------		
mutex               构造函数
----------------------------------------------------------------------		
lock                互斥锁上锁
----------------------------------------------------------------------		
try_lock            如果互斥锁没有上锁，则努力上锁，但不阻塞
----------------------------------------------------------------------		
try_lock_for        如果互斥锁没有上锁，则努力一段时间上锁，这段时间内阻塞，过了这段时间就退出。
----------------------------------------------------------------------		
try_lock_until      努力上锁，知道某个时间点，时间点到达之前将一直阻塞
----------------------------------------------------------------------		
native_handle       得到本地互斥锁句柄
----------------------------------------------------------------------		
```


### 用非阻塞上锁版本改写上例
```cpp
#include <iostream>  // std::cout
#include <thread>    // std::thread
#include <mutex>     // std::mutex
using namespace std;

volatile int counter(0);  //定义一个全局变量，当作计数器，用于累加
std::mutex mtx; //用于包含 counter 的互斥锁

void thrfunc()
{
	for(int i=0;i<10000;++i)
	{
		// 互斥锁上锁
		if(mtx.try_lock())
		{
			++counter;  // 计数器累加
			mtx.unlock(); // 互斥锁解锁 
		}
		else
		{
			cout <<"try_lock false"<<endl;
		}
		
	}
}

int main(int argc, const char* argv[])
{
	std::thread threads[10];

	for(int i=0;i<10;++i)
	{
		threads[i] = std::thread(thrfunc); // 启动10个线程
	}

	for(auto & th:threads)
	{
		th.join();//等待10个线程结束
	}

	cout <<"count to "<<counter<<" successfully "<<endl;

	return 0;
}
```


## 线程池 

```txt
线程池的定义：
  这里的池是形象的说法。线程池就是有一堆已经创建好了的线程，
初始都处于空闲等待状态，当有新的任务需要处理的时候，就从这堆线程
（这堆线程比喻为线程池）中取一个空闲等待的线程来处理该任务，
当任务处理完毕后，就再次把该线程放回池中（一般
就是将线程状态置为空闲），以供后面的任务继续使用。
当池子里的线程全部都处于忙碌状态时，
线程池中没有可用的空闲等待线程，
此时根据需要选择创建一个新的线程并置入池中，
或者通知任务当前线程池里所有线程都在忙，
等待片刻再尝试。


使用线程池的原因：
  线程的创建和销毁相对于进程的创建和销毁来说是轻量级的
（开销没有进程那么大），但是当我们的任务需要进行大量线程的创建
和销毁操作时，这些开销合在一起就比较大了。
比如，当你设计一个压力性能测试框架的时候，需要连续产生大量的并发操作。线程池在这种场合是非常适用的。
线程池的好处就在于线程复用，某个线程在处理完一个任务后，
可以继续处理下一个任务，而不用销毁后再创建，
这样就可以避免无谓的开销，因此尤其适用于连续产生大量并发任务的场合。 
```

### 用C++实现一个简单的线程池


















