

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










