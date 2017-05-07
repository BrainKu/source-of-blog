title: "《深入理解C指针》 >= Ch6"
date: 2014-09-17 23:34:46
tags: [读书,笔记]
---
Chap 6 指针和结构体
=====
**字节对齐**
------
结构体尤其要注意的问题是：**字节对齐**
> Data alignment means putting the data at a memory offset equal to some multiple of the word size, which increases the system's performance due to the way the CPU handles memory --来自WIKI

<!--more-->
```c
// 一定使用typedef才可以在后面sizeof中直接使用Node
typedef struct _node {
	char* content;
	char* test;
	int value;
	short s1;
	short s2;
} Node;

int main() {
	printf("%d\n", sizeof(Node)); // 24
}
```
在我64位的机器上输出是24。~~我老是忘记只有指针的大小跟机器的位数有关- -。~~  
两个 `char*` 16字节，`int` 4字节， 两个 `short` 4字节。去掉所有的short，也还是24字节。  
实际上是按照结构体中最大的单元的n倍来分配空间的。
上面最大的**结构单元**是8字节的指针，那么结构体占有的空间只能是8的倍数。  
如果一个结构体中包含另外一个结构体，那么最小的 **结构单元** 是另外一个结构体与当前结构体中合起来的所有成员中最占内存的单位（而不是把被包含的结构体作为一个单位）
再看一个对齐的例子：
```c
typedef struct node {
	char a[10]; 
	int value;
	char gender;
} Node; //20
typedef struct node { 
	int value;
	char gender;
	char a[10];
} Node; //16
```
输出都写在上面了。当把小单位的单元放在一起时不先考虑packed的问题，小的东西先挤挤看~后面那种情况可以挤一下，于是变成 `char[11]` 不到12字节，就填成12字节，最后也就形成了16字节的空间。  
而上面那种情况，是由于int在中间隔断了两个char,导致虽然两个char相邻，但也只能被分别packed~最后就是20字节啦。

**创建与释放**
----
创建不提，结构体比较容易出现的问题是在释放的时候。经常会出现忘记释放分配的内存的情况，小心谨慎即可。

关于结构体成员变量的访问，如果是结构体指针直接用 `->` 即可，如果是结构体的一个变量，就用点操作符。  
为减少 `malloc` 和 `free` 的开销，可以适当地使用缓存池~  
简单的思路就是：
1. 在 `get` 时查看表中是否存在对象，有则返回，并将该位置置为NULL, 否则创建新对象再返回。 
2. 用完对象之后，调用类似于 `release` 的方法。 此时看缓存池里面有没有空位，有空位就塞进去，没有就直接释放。

Chap 7 安全问题和指针误用
==========
7.1 指针的声明和初始化
--------
#### **不恰当的指针声明**
```c
int* ptr1, ptr2; // 声明了两个变量，前面一个整型指针，后面一个为整型
int *ptr1, *ptr2; // 正确的写法~或者下面这种
typedef int* PINT
PINT ptr1, ptr2;
```
#### **使用指针前未初始化**
初始化之前就使用指针会导致运行时错误，这种指针也被称为 **野指针**  
```c
int *pi; 
// 这里可能可以打印出什么脏数据
// 但也可能因为能够表示的地址不合法而让程序直接崩掉
printf("%d", *pi); 
```

7.2 指针的使用问题
------
可能导致缓冲区溢出的情况：
1. 访问数据时不考虑边界
2. 对数组指针进行算数运算导致越界
3. 使用gets之类的函数从输入中读取字符串
4.**误用strcpy和strcat这样的函数**

#### **错误的解引用**
```c
int num;
int *pi;
*pi = &num; //这里是将num的地址赋给了pi所对应的内存（的值），但是pi还未初始化- -
int* pi = &num; //正确。*代表的时候指针，不要跟解引用搞混
```

#### **越界访问**
```c
char firstname[] = "1234567";
char middlename[] = "1234567";
char lastname[] = "1234567";

middlename[-2] = 'X';
middlename[0] = 'X';
middlename[10] = 'X';

printf("%p %s\n", firstname, firstname);
printf("%p %s\n", middlename, middlename);
printf("%p %s\n", lastname, lastname);
```
预计的输出应该是，lastname的地址最大，firstname的地址最小，且firstname跟lastname都有一个数被修改。
但实际上devC++输出是：
```c
000000000023FE40 1234567
000000000023FE30 X234567
000000000023FE20 1234567
```
应该是有做防缓冲区溢出的检测吧。

#### **错误计算数组长度**
尽量地少用 `strcpy` 这样不需要长度信息的函数。

#### **错误使用sizeof**
*对于数组指针而言，sizeof返回的是字节数，而非数组元素的个数*
另外在作为函数传递的时候，数组指针可能会退化为字符串指针，这个时候的sizeof就只是指针所占的空间而已了。无论参数呈现什么形式：
```c
void test(char test[]) { // 我的机器这里是8字节
// void test(char* test) { // 同上 
	printf("%d\n", sizeof(test));
}
```
#### **不匹配的指针类型**
是什么指针，就用对应的内存。比如short类型的指针，如果把int类型的指针赋给它，它也只能用2字节。

#### **有界指针** ####
~~C中没有智能指针，所以基本没法控制~~

#### **字符串的安全** ####
慎用单纯的`strcpy`。最好用`scan_s`来防止缓冲区溢出。

#### **指针算数运算和结构体** ####
由于`data structure padding` 的存在，对指针的算数运算可能会错误地访问到这些内容。

#### **函数指针** ####
在c中，以下几种使用函数的方式都不会发出警告
```c
int getNumber() {  return 0;  }

int main() {
	cout << getNumber << endl;
	cout << (getNumber == 0)<< endl;
	cout << (getNumber() == 0) << endl;
}
//1 0 1
```
> 另外关于书中提到的函数指针类型参数不匹配但是仍然能够通过编译的现象在我的编译环境上并不存在。在GCC上并不能编译通过，会提示参数类型不匹配。

7.3 内存释放问题
------------------
#### **避免重复释放** ####
在使用完指针之后将其置为`NULL`

#### **清除敏感数据** ####
敏感数据，用完马上复写。`memset(ptr, 0, size)`是个不错的选择。

关于第八章
---------------
讲到了关于**转换指针（将数值强转为地址）**以及**别名**，和**如何用函数指针实现回调**和**如何用结构体，函数指针设计简单的面向对象**。这块内容不做过多讨论~

### 一点小补充 ###
突然间发现，原来C是没有bool类型的啊orz  
也不支持重载。