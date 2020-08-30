

### C++基础知识

- > 函数指针
```cpp
#include <iostream>
using namespace std;
int addition(int a, int b) {
	return (a + b);
}

int subtraction(int a, int b) {
	return (a - b);
}
int(*myf)(int, int)  = subtraction;

int operation(int x, int y, int(*functocall)(int, int)) {
	int g;
	g = (*functocall)(x, y);
	return (g);
}

int main() {
	int m, n;
	m = operation(7, 5, addition);
	n = operation(20, m, myf);
	cout << n << endl;
	return 0;
}
```

```txt
int(*myf)(int, int) 
定义了一个名称是myf的函数指针，返回类型是int,带有2个int 参数。
myf（函数指针）也可以称之为回调函数
```


- > 操作符重载
```cpp
#include <iostream>
using namespace std;
   
class CVector {
public:
	int x, y;
	CVector() {x=0;y=0;};
	CVector(int, int);
	CVector operator +(CVector);
};
    
CVector::CVector(int a, int b) {
	x = a;
	y = b;
}
    
CVector CVector::operator+(CVector param) {
	CVector temp;
	temp.x = x + param.x;
	temp.y = y + param.y;
	return (temp);
}
    
int main() {
	CVector a(3, 1);
	CVector b(1, 2);
	CVector c;
	c = a + b;
	cout << c.x << "," << c.y<<endl;
	return 0;
}  
```



- > try catch 
```cpp
#include <iostream>
using namespace std;
int main() {
	char myarray[10];
	try {
		for (int n = 0; n <= 10; n++) {
			if (n > 9) throw "Out of range";
			myarray[n] = 'z';
		}
	}
	catch (char * str) {
		cout << "Exception: " << str << endl;
	}
	return 0;
}   	
```
  catch 的是throw 的内容

```cpp
#include <iostream>
using namespace std;
int main() {
	try {
		char * mystring;
		mystring = new char[10];
		if (mystring == NULL) throw "Allocation failure";
		for (int n = 0; n <= 100; n++) {
			if (n > 9) throw n;
			mystring[n] = 'z';
		}
	}
	catch (int i) {
		cout << "Exception: ";
		cout << "index " << i << " is out of range" << endl;
	}
	catch (char * str) {
		cout << "Exception: " << str << endl;
	}
	return 0;
}   
```

```cpp
//捕获所有的异常，用三个点表示
catch(...)
{
   cout<<"Exception occurred"<<endl;
}

```

- > 捕获标准的错误
```cpp
#include <iostream>
using namespace std;
#include <exception>
#include <typeinfo>
class A
{
	virtual void f(){};
};


int main()
{
	try{
		A* a = NULL;
		typeid(*a);		
	}
	catch(std::exception& e)
	{
		cout << "Exception: " << e.whatA() <<endl; 
	}

	return 0;
}
```

- > 标准宏

__LINE__  整数值，表示当前正在编译的行在源文件中的行数

__FILE__  字符串，表示被编译的源文件的文件名

__DATE__  一个格式为"Mmm dd yyyy"的字符串，存储编译开始的日期

__TIME__  一个格式为"hh:m:ss"的字符串，存储编译开始的日期

__cplusplus  整数值，大于或等于199711L


```cpp
#include <iostream>
using namespace std;
//标准宏名称
#include <iostream>
using namespace std;

int main()
{
	cout << "This is the line number " 
	     << __LINE__;
	cout << " of file " << __FILE__ 
	     << ".\n";
	cout << "Its compilation began " 
	     << __DATE__;
	cout << " at " << __TIME__ << ".\n";
	cout << "The compiler gives a "
	     << "__cplusplus value of " 
	     << __cplusplus;
	cout << endl;
	return 0;
}
```


- > 用宏定义，来实现Linux下面的日志打印，非常便捷

```cpp
#include <iostream>
using namespace std;
#include <iostream>
#include <string>
using namespace std;
#define DBGDUMP(...) \
{\
    printf("FILE:%s,func:%s,Line%d: ", __FILE__, __func__, __LINE__);\
    printf(__VA_ARGS__);\
}

void test()
{
    cout << "This is the line number " 
         << __LINE__;
    cout << " of file " << __FILE__ 
         << ".\n";
    cout << "Its compilation began " 
         << __DATE__;
    cout << " at " << __TIME__ << ".\n";
    cout << "The compiler gives a "
         << "__cplusplus value of " 
         << __cplusplus<<endl;
    cout <<"FILE:"<<__FILE__<<endl;

    cout <<"function name:"<<__func__<<endl;
}

int main()
{
    int ret = 1;
    DBGDUMP("ret=%d \r\n", ret);  //日志打印输出

    int a=666,b=777;
    string strC = "henry";
    DBGDUMP("a=%d,b=%d,strC:%s \r\n",a,b,strC.c_str());  //日志打印输出，非常便捷

    test();
    cout << endl;
    return 0;
}
```

- > 查看Linux系统字符集的4种方式
```bash
[henry@localhost ~]$ echo $LANG
en_US.UTF-8

[henry@localhost ~]$ env | grep LANG
LANG=en_US.UTF-8

[henry@localhost ~]$ export | grep LANG
declare -x LANG="en_US.UTF-8"

[henry@localhost ~]$ locale 
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
[henry@localhost ~]$ 
```


- > 修改Linux系统字符集
```bash

方法1：
export  LANG=zh_CN.UTF-8


```









