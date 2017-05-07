title: 《深入理解C指针》 Ch5
date: 2014-09-17 20:34:31
tags: [笔记,读书]
---
Chap5 指针和字符串
======
**5.1 基础**
------------
> 字符常量时单引号引起来的字符序列。字符常量通常由一个字符组成，也可包含多个字符（如转义）。在C中，字符常量是**int类型**。

声明字符串的方式有三种: 字面量，字符数组，字符指针。  
字符串字面量是用双引号包裹的字符序列，它位于字符串字面量池中。
**！**跟单引号的字符字面量不同  
<!--more-->

### **字符串字面量池**
定义字面量时通常会将其分配在字面量池中，这个区域保存了组成字符串的字符序列。多次用到同一个字面量时，字面量池通常也只有一份副本。
> **通常**情况下，字符串字面量是不可修改的（不然会导致指向这个字符串字面常量的所有引用数值都改变）。但你不能保证编译器会告诉你。  
> 下面这段代码，在DevC++，使用GCC可以编译通过，但是在实际执行时会报段错误。  
> **建议将这种指向字符串字面量的指针设置为const，如此就算一不小心修改了，也能够在编译时得到错误。将失败提前是一种有效避免bug的方式**
>```c
char *a = "hello world";
// const char *a = "hello world"; 建议
*a = 'L'; 
printf("%s", a);
```

### **字符串初始化**
字符串所对应的内存要么是一块内存，要么是数组。  
几种初始化方式：
```c
char header[] = "hello world"; //将常量池中的内容复制到数组中（注意长度总是要比strlen + 1('/0')
char header1[12];strcpy(header1,"hello world");
char *header2 = (char*)malloc(12*sizeof(char));strcpy(header2,"hello world");
char *header3 = "hello world"; //直接将字符串字面量的地址赋给header3--也就是指向唯一一个常量。
```
> 上面有种初始化方式是将字符串字面量直接赋给字符串指针。那么如果字符字面量又如何？
```c
// char *c = 'a'; 直接报错。上面提到过，字符字面量是int类型，上述行为就相当于将整数赋给字符指针。
// 在解引用的时候就会报错了。这里的错误就是相当于，未初始化变量导致的错误= =
char *c = (char*)malloc(1); *c = '+';
// 这样子是合法的，因为'+'并不会超过128，也就不会溢出。两者都分配了空间，也就能够正常调用了。
char c[1]; c[0] = 'a'; 
```

### **位置**
全局指针，静态变量指针的字符串变量均指向字符串字面量。  
数组则是对应的空间-比如全局和静态都在全局内存，其他在栈中，或者堆中。

**5.2 字符操作**
-----
`int strcmp(const char* s1, const char* s2)`  
如果小于0，则代表s1<s2,其他类推（这么想就好了s1-s2<0)  
`char* strcpy(char* dst, const char* from)`  
复制指定字符串到特定的地址。返回的是指向'\0'前一个位置的地址（最好不要用返回值） 
> 默认的实现是需要先为dst分配空间再传入的。稍微实现一个传入未分配空间的字符串指针的拷贝函数：
```c
char* my_strcpy(char* dst, char* src) {
	dst = (char*) malloc(strlen(src) + 1);
	char* tmp = dst;
	while (*dst++ = *src++);
	return tmp; //如果返回了指针，记得在使用完后释放内存free~
}
```

`char* strcat(char* dst, const char* from)`  
dst必须有足够的内存，否则会覆盖。函数返回拼接后字符串的首地址,也就是dst。注意前面的'\0'会被清除。

**5.5 函数指针和字符串**
---------------
将字符转为小写的函数：  
【感觉我有严重的误解= =const char\*代表不可以通过指针修改值，但是指针可以指向其他值。**它认为它自己指向的是常量** 】
```c
char* stringToLower(const char* string) {
	char* tmp = (char*) malloc(strlen(string)+1);
	char* start = tmp;
	while (*string != 0) {
		*tmp++ = tolower(*string++);
	}
	*tmp = 0;
	return start;
}
```
原来这样子可以创建函数指针类型：
```c
typedef int (name*)(const char*, const char*);// return type, function name, params
```

测试小题目
-----------
> 9-18：突然间有个疑问，觉得可以作为一个问题来问下~刚好跟这章内容有关~

考虑以下代码的输出：
```c
int main() {
	char* a = "arr";
	char  b[] = "arr";
	printf("%d\n", a == "arr");
	printf("%d\n", b == "arr");
}
```
解答：==在c里面的含义还是一般意义上相等，也就是比较数值。这里比较的都是指针的地址，`"arr"` 是一串分配在字面量常量池中的字面量，这里将它理解成一个指向这个字面量的常量指针。那么很明显地可以得出答案了吧~数组b是分配在栈上的，地址明显跟常量池的 `"arr"` 不同。而关于 a 常量池中同一字符串通常只有一份副本。所以最后的输出是；
```c
1
0
```