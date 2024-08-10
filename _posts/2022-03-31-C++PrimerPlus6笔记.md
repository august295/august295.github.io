---
layout: post
title: "C++PrimerPlus6笔记"
categories: Book
tags: Book
author: August
typora-root-url: ..
---

* content
{:toc}


该文记录《C++ Primer Plus（第六版）》读后记录。



# C++ Primer Plus 



## 1. 预备知识

Preparatory knowledge

### 1.1 C 与 C++ 的区别与联系

C++ 是 C 的超集，包含 C 而且衍生特性。

C   结构化编程（Structurd Programming, SP）

   - 包含控制结构、函数，以便更好地控制流程，
   - 结构化、模块化

C++ 面向对象编程（Object-Oriented Programming, OOP）

   - 面向对象，泛型编程
   - 模块化，重用代码



### 1.2 标准定义

美国国家标准局（America National Standards Institute, ANSI）

国际标准组织（International Standards Organization, ISO）

```
C   标准 C99
C++ 标准 C++98 C++11
```



### 1.3 代码编译过程

```
代码编译过程 
.cpp  ->  .o  ->  .out/.exe
编译 
链接 

g++ -std=c++98 *.cpp 
```



## 2. 开始学习 C++

### 2.1 认识

```
程序必须包含 `main()` 函数，程序从 main 开始执行

注释使用 
	// 		是单行注释
	/**/	是多行注释（C 注释风格）
	
头文件
	C++ 头文件都是与 C 语言有差异，标准是 <iostream> 不带 .h 后缀，但是可以兼容
	C++ 还将 C 语言标准头文件更改，例如 <math.h> <cmath>
	
名称空间
	using namespace std;	std 相当于标准对象，直接可以使用其中函数，例如 std::cout

cout 格式化输出
```

| 流操纵算子 | 作  用                                                                        |      |
| ---------- | ----------------------------------------------------------------------------- | ---- |
| dec        | 以十进制形式输出整数                                                          | 常用 |
| hex        | 以十六进制形式输出整数                                                        |      |
| oct        | 以八进制形式输出整数                                                          |      |
| fixed      | 以普通小数形式输出浮点数                                                      |      |
| scientific | 以科学计数法形式输出浮点数                                                    |      |
| left       | 左对齐，即在宽度不足时将填充字符添加到右边                                    |      |
| right      | 右对齐，即在宽度不足时将填充字符添加到左边                                    |      |
| setbase(b) | 设置输出整数时的进制，b=8、10 或 16                                           |      |
| setw(w)    | 指定输出宽度为 w 个字符，或输人字符串时读入 w 个字符                          |      |
| setfill(c) | 在指定输出宽度的情况下，输出的宽度不足时用字符 c 填充（默认情况是用空格填充） |      |



### 2.2 C++ 语句

```
C++ 
	变量必须提前声明变量类型
	= 赋值语句
cout
cin
```

运算符优先级

相同优先级中，按结合顺序计算。

<font color = red>大多数运算是从左至右计算，只有三个优先级是从右至左结合的，它们是单目运算符、条件运算符、赋值运算符。</font>

基本的优先级需要记住：

+ 指针最优，单目运算优于双目运算。如正负号。
+ 先乘除（模），后加减。
+ 先算术运算，后移位运算，最后位运算。



### 2.3 函数

函数特性

+ 函数头，函数体
+ 参数
+ 返回值
+ 函数原型



### 2.4 杂记

（1）命名空间

```
使用方法
	1. 放在最前面
		using namspace std;
	2. 放在特定函数中
		void function() { using namspace std; ...... }
	3. 特定函数使用特定方法
		using std::cout;
	4. 不定义，完全路径对应
		std::cout << "Hello World!" << std::endl;
```

（2）风格

符合开发团队即可



## 3. 处理数据

### 3.1 变量

（1）变量格式

```c++
int number = 10;
```

+ 数据类型
+ 变量名
+ 数据数值

名称中只能使用 字母字符、数字、下划线(_)

`int` 型是计算机使用最自然的数（运算最快）

### 3.2 变量分类

#### 3.2.1 整型

（1）只表示有符号的（最高位是符号位），无符号的不要负数部分就是

只是大概取法，根据系统的位数实际情况不相同。

| 类型      | 范围（bit）  |
| --------- | ------------ |
| char      | 8            |
| short     | 16           |
| int       | 32           |
| long      | 64           |
| long long | 128          |
| wchar_t   | 应对 Unicode |
| char16_t  | 16           |
| char32_t  | 32           |
| bool      | 1            |

8 bit = 1 byte（8 比特 等于 1 字节）

可以通过 `<limits.h>` 或 `<climits>` 进行验证

+ MAX
+ sizeof()
+ CHAR_BIT

（2）整型字面值（表示）

C++ 默认显示十进制，需要设置才能显示其他格式

```
八进制      oct		043
十进制      dec		43
十六进制    hex		0xAA
    
int num = 15;
cout << hex;
cout << num;
```

（3）确定常量类型

一般存储在 int 中，超出范围或后缀除外


（4）转义字符

```
\
```

（5）const 常量声明



#### 3.2.2 浮点型

（1）科学记数法

```
+5.37E+16
```

（2）浮点类型

| 类型        | 范围      |
| ----------- | --------- |
| float       | 32        |
| double      | 64        |
| long double | 80 96 128 |

（3）后缀表示

建议后缀全部大写

```
u/U		表示 unsigned int 
l/L		表示 unsigned long
f/F		表示 float

例如
u U ull ULL f F
```



### 3.3 算数运算符

+ 明白算数优先级
+ 运算会提升到同一个级别计算


（1）类型强制转换

```c++
char n = 65;

(int) n;				// old C
int (n);				// new C++
static_cast<int>(n);	// C++
```

（2）auto

+ 自动确定变量类型
+ 在使用迭代器时十分方便



## 4. 复合类型

### 4.1 数组

+ 存储值类型
+ 数组名
+ 元素数

```c++
typeName arrayName[arraySize]
int num[10] = {1, 2, 3};
int num[] = {1, 2, 3};
int num[10] = {};
```

初始化，未实际赋值的默认全部是0；

### 4.2 字符串

+ 长度计算一定要包含截止符 `\0`
+ `''` 表示单个字符，`""`表示字符串

```
char str[10] = {'a', 'b', 'c'};
char ch = 'S';
char *str = "hello world";
```

拼接字符串 `"" ""` 直接拼接

### 4.3 string 类

个人认为 string 和 char* 或 char 数组一样

```
char c1[20];

string s1 = "parent";
string s2;
s1 = s2;

<cstring>	// C <string.h>
可以实现字符串的复制
```

+ `+` 重载拼接，可以自动调整大小



原始字符串（raw），显示字符串的原始数据，不进行转义

```
R"( )"
可以自定义 "( 与 )" 之间的标识符，例如： R"+*( )+*" 
```



### 4.4 结构

#### 4.4.1 结构体（struct）

```c++
struct people
{
	int age;
	string name;
};
```

+ C++ 声明时允许省略关键字 struct
+ 成员运算符 `.`

位字段

```c++
struct bits
{
	unsigned int SN : 4;	// SN 占用 4 bits
    unsigned int : 4;		// 不使用 4 bits
    bool in : 1;			// in 占用 1 bits
    bool out : 1;			// out 占用 1 bits
};
```



#### 4.4.2 共用体（union）

+ 能够存储不同类型的数据类型，但是只能同时存储一种类型
+ 长度为共用体最大成员长度

```c++
union val
{
	char char_val;
	int int_val;
	long long_val;
	double double_val;
};
```

+ 匿名共用体：常用于结构体内部，被视为对象的两个成员，地址相同，不需要标志，只负责哪个成员活动

```c++
union
{
	long id_num;
    char id_char[20];
};
```



### 4.5 枚举（enum）

+ 默认开始是 0，依次加 1。
+ 如果有设置，后面没有被初始化的比前面的大1
+ 枚举范围 $2^n$



### 4.6 指针

+ 指针是一个变量，存储的是值得地址
+ （&）地址运算符 
+ （*）间接值，解除引用运算符

```
int* p1, p2;
这是 int *p1 和 int p2
```

new 和 delete 尽量同时使用且在同一块区域

```
new		delete
new[]	delete[]
```



### 4.7 指针 数组 指针运算

+ 指针可以取数组开始位置地址

+ 指针加1，代表指针地址加上数据类型所占字节

```
例如：
int *n = 1;
&n = 0x121200;
n++;
&n = 0x121204
```

指针和数组

```
pn[0] = *pn;
pn[1] = *(pn + 1);
```

+ （->）箭头成员运算符，指针成员运算符

```c++
struct number * pn;
(*pn).price    <===>   pn->price
```



### 4.8 数组替代品

```c++
// 可动态调整
C++98	vector	
vector<int> a(10);

// 定长度数组，更安全
C++11	array
array<int, 10> a;
```



## 5. 循环和关系表达式

### 5.1 循环

+ for

```
for ( 1; 2; 4 )
{
	3
}

1	initialization		设置值初始值
2	test-expression		执行测试表达式
3	statement			执行循环操作
4	update-expression	更新测试值表达式
    
判断条件默认 0 是 false，其他是 true，可以用来判断截止符 '\0'
    for(;;){} 将会一直循环
```



+ while：下判断后执行

```
while( test-expression )
{
	statement
}
相当于
for ( ; test-expression ; ){
	statement
}
```



+ do while：先执行后判断

```
do {
	statement
} while( test-expression )
```



### 5.2 表达式

+ 自增运算符

```
分为前缀和后缀，结果都会把自己增加1
	前缀（效率高）：
		先自增1，再做运算
		直接改变自身
	后缀：
		先做运算，再自增1
		复制一个副本，副本加1，返回副本
i++
++i

i--
--i
```



+ 组合赋值运算符

| 操作符 | 含义（L += R;） |
| ------ | --------------- |
| +=     | L = L + R;      |
| -=     | L = L - R;      |
| *=     | L = L * R;      |
| /=     | L = L / R;      |
| %=     | L = L % R;      |



+ 关系运算符

| 操作符 | 含义     |
| ------ | -------- |
| <      | 小于     |
| <=     | 小于等于 |
| ==     | 等于     |
| >      | 大于     |
| >=     | 大于等于 |
| !=     | 不等于   |



+ 逗号运算符

表示将两个语句分隔开，都可以执行



### 5.3 杂记

+ typedef

可以起别名，比 `#define` 直接替代更好

+ cin

```
cin.get()
cin.getline()
```

+ 文件尾

```
EOF
```



## 6. 分支语句和逻辑运算符

### 6.1 分支语句

+ if

```c++
if ( test-condition ) {
	stetament
}
```

+ if else

```
if ( test-condition ) {
	stetament
} else {
	statement
}
```

+ if    else if    else

```
if ( test-condition ) {
	stetament
} else if ( test-condition ) {
	statement
} else {
	statement
}
```

+ switch

```c++
switch (integer-expression) {
	case lable1: statement; break;
	case lable2: statement; break;
	case lable3: statement; break;
	default: statement; break;
}
```

如果没有 break，就会一直顺序执行

`switch` 判断的是数字，字符也是转换成 `ASCII` 进行判断

当判断条件超过 3 建议使用 switch

+ break 和 continue

```
break		是直接跳出当前循环
continue	是跳过循环中后面语句，重新判断条件
```





### 6.2 运算符

+ 逻辑运算符

| 操作符 | 含义   |
| ------ | ------ |
| &&     | 与 and |
| \|\|   | 或 or  |
| !      | 非 not |

+ `? :` 条件运算符（三目运算符）

```
n = n > 5 ? 1 : 0
当满足条件 n 得到的值是 1，不满足得到的是 0
```



### 6.3 ctype  字符处理头文件

| 单字节   | 宽字节    | 描述                                            |
| -------- | --------- | ----------------------------------------------- |
| isalnum  | iswalnum  | 是否为字母数字                                  |
| isalpha  | iswalpha  | 是否为字母                                      |
| islower  | iswlower  | 是否为小写字母                                  |
| isupper  | iswupper  | 是否为大写字母                                  |
| isdigit  | iswdigit  | 是否为数字                                      |
| isxdigit | iswxdigit | 是否为16进制数字                                |
| iscntrl  | iswcntrl  | 是否为控制字符                                  |
| isgraph  | iswgraph  | 是否为图形字符（例如，空格、控制字符都不是）    |
| isspace  | iswspace  | 是否为空格字符（包括制表符、回车符、换行符等）  |
| isblank  | iswblank  | 是否为空白字符(C99/C++11新增)（包括水平制表符） |
| iswprint | iswprint  | 是否为可打印字符                                |
| ispunct  | iswpunct  | 是否为标点                                      |
| tolower  | towlower  | 转换为小写                                      |
| toupper  | towupper  | 转换为大写                                      |



### 6.4 文件操作

```c++
#include <fstream>	// 头文件

ofstream	// 写入文件，同 cout
    ofstream outFile;
	outFile.open(filePath);
	outFile << "you data" << endl;
	outFile.close();
    
ifstream	// 读取文件，同 cin
    ifstream inFile;
	inFile.open(filePath);
	inFile >> "file data";	// inFile.getline()
	inFile.close()
```



+ char* 和 string 读取使用

```
char 可以指定长度
	char name[32];
	cin.getline(name, 32);

string 则不同，需要指定获取
	string name;
	getline(cin, name);
	
都可以重载截断字符
	cin.getline(name, 32, '#');
	getline(cin, name, '#');
```



## 7. 函数——C++的编程模块

### 7.1 函数的基本知识

（1）函数使用

```
1. 提供函数定义
	int max(int a, int b) {
		return a > b ? a : b;
	}
2. 提供函数原型
	int max(int a, int b);
	int max(int, int);
3. 函数调用
	max(10, 5);
```

+ 函数原型相当于一个接口
+ 函数原型不要求提供变量名，变量名相当于占位符，变量名会使函数传递参数更容易理解



### 7.2 函数和数组

#### 7.2.1 数组传参

```
在 C++ 和 C 中，将数组视作为指针
    数组传递的是数组名，是第一个元素的地址
	int arr[] 相当于 int  *arr
    
当要传递数组，一定要传递数组和长度，不能使用方括号表示
    void max(int arr[], int size);	// OK
	void max(int arr[size]);		// NO

数组区间传递，一定要按正确顺序传递且指向正确地址
    int sum(const int *begin, const int *end);
```

#### 7.2.2 const 

```
1. 使用
对于不会修改的数据，最好使用 const 传递，保证接下来不会对数据产生修改
const int x = 10;	// 定义常量

2. 指针和const
如果数据本身不是指针，可以将数据地址赋值给 const 指针，但只能将 非const 数据赋值给 非const 指针
    int a = 1;
    const int b = 2;
    int *p1 = &a;		// 合法
    const int *p2 = &b;	// 合法
    int *p3 = &b;		// 不合法
    const int *p4 = &b;	// 合法

3. const int 和 const pointer
    int a = 1;
	int b = 2;
	const int * p1 = &a;	// 指针指向 int 常量
	int * const p2 = &a;	// 常量指针指向 int 
p1 不能修改指向的值，但是可以修改 p1 的地址
    p1 = &b;
p2 可以修改指向的值，但不允许修改 p2 的地址
    *p2 = 10;
```



### 7.3 函数和二维数组

```c++
int *a[4] 	// 指针数组，4个int型指针
int (*a)[4] // 数组指针，指向的数组列数为4
int a[][4]	// 相当于 (*a)[4]

a[r][c] == *(*(a + r) + c) // 二位数组取值
```



### 7.4 C 语言风格字符串

传递的是第一个字符的地址

数组一定要多预留一个位置，保存截止符

```c
char *str1 = "12345";
char str2[10] = "12345";
```



###  7.5 函数传递

成员运算符（.），间接成员运算符（->）

```
1. 值传递
	不能修改传入参数的数据
	
2. 地址传递
	可以修改传入参数的数据
	如果不可修改，可用 const 进行定义 
```



### 7.6 递归

```
函数递归调用自己
每个递归调用都会创建自己的一套变量

多个递归有时被称为分而治之策略（divide-and-conquer strategy）
```



### 7.7 函数指针

```
1. 获取函数的地址
2. 声明一个函数指针
3. 使用函数指针调用函数
```

同一原型，数组就是指针表示

```c++
double *f1(int arr[], int n);
double *f2(int [], int);
double *f3(int *, int);
```

auto 能够极大简化**自动**推断单值类型

```c++
double *(*p1)(int *, int) = f1;
auto p1 = f1;
```



typedef 起别名也能简化声明

```c++
typdef double *(*p_fun)(int *, int);
p_fun p1 = f1;
```





## 8. 函数探幽

### 8.1 C++ 内联函数

+ 调用函数

执行到函数调用指令的时候，程序将存储该指令的内存地址，并将函数参数复制到堆栈，跳到函数执行的内存单元，执行函数代码，然后跳回到地址被保存的指令处



+ 内联函数

```
inline
    在函数声明前加上关键字 inline
    在函数定义前加上关键字 inline
通常的做法是：省略函数原型，将整个定义放在提供原性的地方
    inline double square(double x) { return x * x; }
```

内联函数就是将函数直接编译是放在代码中，不需要在跳转，占用更过内存空间，但节约运行跳转时间



+ 宏函数

仅仅是实现替换，只能使用固定值，不能使用可改变值（n++）



### 8.2 引用变量

#### 8.2.1 引用基本概念

C++ 赋予 `&` 引用的含义

```c++
int a = 10;
int *x = &a;		// 取地址 
int &x = a;			// 引用
int * const x = a;	// 引用相当于固定地址的指针（常量指针）
```

+ 引用声明时，必须初始化
+ 引用数据修改，原来数据也会修改（常用于函数参数修改）

```
传递参数修改
	使用引用
	使用指针
```

+ 使用时，和普通传参一样，只是在函数定义和函数原型时不一样

```c++
void swap(int x, int y);	// swap(a, b)
void swapr(int &x, int &y);	// swapr(a, b)
void swapp(int *x, int *y);	// swapp(&a, &b)
```

+ 引用参数、临时变量、const

```
1. 实参与引用参数类型不匹配会自动创建正确临时变量，进行计算
    实参类型正确，但不是左值
    实参类型不正确，但是可以转换为正确的类型
```

+ 函数返回值引用

```
如果函数直接返回数据，首先会将数据复制到一个临时位置，再将这个拷贝给新定义数据
返回引用是直接复制给新定义数据，效率更高
```



#### 8.2.2 应用参数使用条件

```
引用参数使用原因
	能够修改传入的数据对象
	通过传递应用而不是拷贝整个数据对象，提高运行效率
```

+ 值不做修改

| 条件         | 参数传递方式 |
| ------------ | ------------ |
| 数据对象很小 | 值传递       |
| 数组         | 指针         |
| 数据对象很大 | 指针、引用   |
| 类对象       | 引用         |

+ 值需要被修改

| 条件               | 参数传递方式 |
| ------------------ | ------------ |
| 数据是内置数据类型 | 指针         |
| 数组               | 指针         |
| 结构               | 指针、引用   |
| 类对象             | 引用         |



### 8.3 默认参数

默认参数表示函数调用，如果不传入参数进行覆盖，就会自动使用设置的默认参数

```c++
// 1. 设置默认值，必须通过函数原型
	void setNum(int a, int b, int c);
// 2. 对于带参列表的函数，必须从右向左添加默认值
    void setNum(int a, int b = 1, int c = 1);	// OK
    void setNum(int a = 1, int b = 1, int c);	// NO
// 3. 实参按从右到左的顺序依次被赋给相应的形参，不能跳过任何参数
    void setNum(1);			// OK, (1, 1, 1)
    void setNum(1, 2, 3);	// OK
	void setNum(1, , 3);	// NO
```



### 8.4 函数重载

函数重载声明几个功能类似的同名函数，但是这些同名函数的形式参数（指参数的个数、类型或者顺序）必须不同，也就是说用同一个函数完成不同的功能

```c++
// 1. 案列
	void print(const char *str);
	void print(const long num);
	void print(const double num);
// 2. 如果没有原型匹配，C++会尝试使用标准类型进行强制转换，如果强转类型唯一（例如：整型），可以执行
    print("1.1");	// void print(const char *str);
	print(1.1);		// void print(const double num);
	print(1);		// 如果没有 void print(const long num); 会调用 void print(const double num); 但是有就会报错
```



### 8.5 函数模板

```c++
// 1. 使用
template <typename T> 或 template <class T>
void swap(T &a, T &b)
{
    T temp = a;
    a = b;
    b = temp;
}

// 2. 可以指定类型
int x = 1, y = 2;
swap(x, y);
swap<int>(x, y);
```



### 8.6 总结

+ 传入参数个数不同使用默认参数
+ 参数类型不同进行不同操作，使用函数重载
+ 参数类型不同进行相同操作，使用函数模板



## 9. 内存模型和名称空间

### 9.1 单独编译

C++建议将组件函数放在独立的头文件中

```
1. 结构
	头文件
	头文件的源代码文件
	源代码文件
2. 头文件内容
	函数原型
    常量
    结构
    类
    模板
    内联
3. 头文件格式
	#ifndef MYHEAD_H_
    #define MYHEAD_H_
    ......
    #endif
4. 头文件引用
	标准头文件使用 <>
	用户自定义头文件使用 ""	
```



### 9.2 存储持续性、作用域、链接性

+ 存储持续性

静态存储持续性：程序运行中都存在，`static`

动态存储持续性：使用 `delete` 进行释放、作用域结束、程序结束



+ 作用域

```
1. global scope(全局作用域符），用法（::name)
2. class scope(类作用域符），用法(class::name)
3. namespace scope(命名空间作用域符），用法(namespace::name)
```



### 9.3 名称空间

变量的潜在作用域从声明开始到声明区域结束

名称空间为了更好的控制作用域，减少名称冲突

```c++
namespace Person {
   	struct Person {
        int age;
        string name;
    };
    void eat();
    void sleep();
}

namespace Animal {
   	struct Animal {
        int age;
        string name;
    };
    void eat();
    void sleep();
}
```

可以使用 `using namespace std`  或者 `using std::cout` 选择使用的名称空间

+ 单个文件可以使用全局名称空间
+ 多文件尽量在作用域内使用名称空间，不然会产生覆盖



## 10. 对象和类

### 10.1 过程性编程和面向对象编程

+ 类

```
1. 定义
	类是一种将抽象转换为用户定义类型的C++工具，它将数据表示和操纵数据的方法组合成一个整洁的包
2. 组成
	类声明：数据成员描述数据，成员函数（方法）描述公有接口
	类方法定义：描述如何实现成员函数
3. 提示
	class Person {}
	成员函数可以就地定义，也可以用原型表示
    权限 private public protected （结构默认是 public 而类默认是 private ）
    作用域解析运算符 ::
	可以在内部使用 inline
    每个创建对象都有自己的存储空间（内部变量和类成员），但同一个类的所有对象共有同一组方法
        
```

类与对象之间的关系同标准类型与其变量之间的关系相同



### 10.2 类的构造函数与析构函数

#### 10.2.1 构造函数

```
1. C++ 提供了一个特殊的类成员函数——类构造函数，将值赋给它的数据成员
	名称与类名相同
	没有声明类型（不是 void ）
2. 成员名和参数名
    参数名（一般）不能与类成员名相同
    	使用（m_）前缀
    	使用（_）后缀
3. 声明和定义构造函数
    一般是充当初始化操作
    1) 默认参数完成 
    Person::Person(const string &name, int grade = 0);
	2) 传入参数完成 
    Person::Person(const string &name, int grade) {
	    m_name = name;
        m_grade = grade;
    }
4. 默认构造函数
    在没有定义任何构造函数时才会自动生成构造函数，有了就不会，但是有时使用隐式声明对象，就需要空的默认构造函数，所以建议任何时候都提供一个默认构造函数
    Person::Person() {}
5. 使用构造函数
    隐式使用
    Person p;
	Person p("Tom", 0);
	显示使用
    Person p = Person("Marry", 1);
	Person *p = new PerSon("Marry", 1);	// 新建对象，指针管理
```



#### 10.2.2 析构函数

```
1. 定义
    析构函数是在类名前加 ~
2. 使用
    在没有定义析构函数时，也会提供一个默认的析构函数
    释放内存
```



### 10.3 this 指针

```
this 指针指向用来调用成员函数的对象（this 被作为隐藏参数传递给方法）
所有的类方法都将 this 指针设置为调用它的对象的地址
如果方法需要引用整个调用对象，则可以使用 *this
```



### 10.4 类作用域

```
1. 作用域
类作用域意味着不能直接从外部访问类的成员
根据上下文确定使用
	直接成员运算符（.）
    间接成员运算符（->）
    作用域解析运算符（::）
    
 2. 类常量
    enum { Months = 12 };
    static const int Months = 12;
```



## 11. 使用类

#### 11.1 运算符重载

```c++
operator+()
operator*()
```

+ 重载必需须是 C++ 有效的运算符，不能虚构一个新的符号
+ 参数申明为引用提高效率，减少内存使用（不修改最好使用 const）
+ 单参数运算符左侧的对象是调用对象，右侧是作为参数传递的对象

```
重载限制
	1. 运算符对应运算操作：不能用加法运算符（-）重载数值相加（+）
	2. 不能违反运算符原来的语法规则，不能修改运算符优先级
	3. 不能创建新的运算符
	4. 不能重载的运算符
		sizeof	
		.		成员运算符
		.*		成员指针运算符
		::		作用域解析运算符
		?:		条件运算符
		typeid	RTTI运算符
		*_cast	强制类型转换运算符
	5. 只能通过成员函数重载的运算符
		=		赋值运算符
		()		函数调用运算符
		[]		下标运算符
		->		指针访问成员运算符
```



### 11.2 友元

通过让函数成为类的友元，可以赋予该函数与类的成员函数相同的访问权限

在为类重载二元运算符时（带有两个参数的运算符）通常使用友元

```c++
friend Time operator*(double m, const Time &t);	// 只有在类声明的原型中才能使用 friend 关键字，实现时和普通函数一样
```

为了实现  `<<` 的链式使用，需要返回引用的 `ostream` 对象

```c++
std::ostream &operator<<(std::ostream &os, const Time &t) {
    os << t.hours << "hours, " << t.minutes << " minutes";
    return os;
}
```



### 11.3 选择成员函数还是非成员函数

```c++
Time operator+(const Time &t) const;
friend Time operator+(const Time &t1, const Time &t2);

time = t1 + t2;
time = t1.operator+(t2);	// 成员函数
time = operator+(t1, t2);	// 友元函数
```



状态成员 `state member` ，记录描述对象所处的状态



### 11.4 类的自动转换和强制类型转换

转换函数

+ 用户定义的强制类型转换

+ 类方法、不能指定放回类型、不能有参数

```c++
Stonwt::operator int() const {
    return int (pounds + 0.5);
}
```



## 12 类和动态内存分配

### 12.1 动态内存和类

```
new 和 delete 动态控制内存
析构函数必不可少
```

#### 12.1.2 特殊成员函数

```
默认构造函数
默认析构函数
复制构造函数
	逐个复制非静态成员（成员复制也称浅复制），复制的是成员的值
赋值运算符
地址运算符
```



```
如果复制的是指针，就只会传递指针首地址，导致两个指针指向同一个地址
解决办法：
	进行深度复制（deep copy），每个对象都有自己的字符串，而不是引用另一个对象的字符串
	浅复制指挥复制指针信息，不会复制指针引用的结构
```



### 12.3 注意事项

```
new 和 delete； new[] 和 delete[]
NULL 0 nullptr
应定义一个复制构造函数，通过深度复制将一个对象初始化为另一个对象
应当定义一个赋值运算符（=），通过深度复制复制给另一个对象
```



### 12.4 返回对象

```
返回对象将调用复制构造函数，二返回引用不会
返回对象是局部变量，不应该使用引用（引用指向同一个地址，但是局部变量函数执行完会自动释放，导致指向错误）
尽量使用引用（引用效率高）
```

 

### 12.5 杂记

+ 类构造函数初始化

```c++
// 只能适用于构造函数
Classy::Classy(int n, int m) : mem1(n), mem2(m), mem3(n * m + 2)
{
    // ...
}

Classy::Classy(int n, int m)
{
    mem1 = n;
    mem2 = m;
    mem3 = n * m + 2;
    // ...
}
```

 

## 13 类继承

继承的作用

+ 可以在已有类的基础上添加新的功能
+ 可以给类添加数据
+ 可以修改类方法

### 13.1 基类

原始类称为基类（父类），继承类称为派生类

```
1. 继承属性
	派生类对象存储了基类的数据成员（继承基类的实现）
	派生类可以使用基类的方法（继承基类的接口）
2. 继承添加特性
	派生类需要自己的构造函数
	派生类可以根据需要添加额外的数据成员和成员函数
```

派生类创建规则

```
1. 如果不调用基类构造函数，派生类将使用基类默认的积累构造函数
2. 复制构造函数使用但没定义，编译器将自动生成一个
3. 类调用顺序
	基类构造函数--> 派生类构造函数 --> 派生类析构函数 --> 基类析构函数
```



### 13.4 静态联编和动态联编

将源代码中的函数调用解释为执行特定函数代码块称为函数名联编

+ 在编译过程中进行联编称为静态联编（早期联编）
+ 编译器生成能够在程序中运行时选择正确的虚函数代码称为动态联编（晚期联编）

```
如果要重新定义基类方法，则将他设置为虚函数
虚函数表：保存一个指向函数地址数组的指针（只添加一个地址成员，表的大小不同）
```

