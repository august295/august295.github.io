---
layout: post
title: "EffectiveC++笔记"
categories: Book
tags: Book
author: August
typora-root-url: ..
---

* content
{:toc}


该文记录《Effective C++（第三版）》读后记录。



# Effective C++（第三版）

改善程序与设计的55个具体做法



## 1 让自己习惯C++

### 01 视 C++ 为一个语言联邦

C++ 相当于多种语言思想的集合，高效编程视状况而变化。

```
1. C
	C++ 是以 C 为基础的。
	区块（blocks）、语句（statements）、预处理器（preprocessor）、内置数据类型（built-in data types）、数组（array）、指针（pointer）等都来自 C。

2. Object-Oriented C++
	就是 C with classes。
	类（classes），封装（encapsulation）、继承（inheritance）、多态（polymorphisms）、虚函数（virtual）等。

3. Template C++
	C++ 泛型编程。

4. STL
	标准模板库。
```



### 02 尽量以 const，enum，inline 替换 define

```
1. 对于单纯常量，最好以 const 或 enum 替换 #define
	原因：可以直观的查看定义

2. 对于形似函数的宏（macross），最好改用 inline 函数替换 #define
	原因：宏仅仅是替换，如果传入参数是前缀自增（或自减）数据会多次改变
	#define MAX(a, b) f((a) > (b) ? (a) : (b))
```



### 03 尽可能使用 const

const 表示 “不可改动” 

```
1. const 可以帮助编译器侦测出错误用法
	可作用于常量、作用域对象、函数参数、函数返回类型、成员函数本体

2. 编译器强制实施 bitwise constness，但是编写程序应该使用“概念上的常量属性”
	可以对属性使用 mutable 来改变 const 函数中变量值
```



### 04 确定对象被使用前已被初始化

```
1. 为内置型对象进行手工初始化
	C++ 初始化对象，数值有时会被初始化为 0，但更多时候是无意义的值
	
2. 构造函数最好使用成员初值列；初值列列出的成员变量其排列次序应该和在 class 中声明的次序相同
	对象构造函数赋值（assignment）和初始化（initialization）不一样；函数赋值先调用默认构造函数为属性设置初值，
	再对他们赋予新值，成员初值列直接调用拷贝构造进行赋初值，比较高效。
	// 赋值
	Person(std::string name, int age) 
	{
		_Name = name;
		_Age = age;
	}
	// 初始化，成员初值列
	Person(std::string name, int age) 
		: _Name(name), _Age(age)
	{}

3. 为免除“跨编译单元初始化次序”问题，以本地静态对象替换非本地静态对象
	FileSystem & tfs()
	{
		static FileSystem fs;
		return fs;
	}
```

 

## 2 构造、析构、赋值运算

### 05 了解 C++ 默默编写并调用哪些函数

一个空类，编译器会为你声明一个默认构造函数、一个拷贝构造函数、一个析构函数、一个赋值构造函数

```c++
class Empty { };

class Empty {
	Empty() {};
	Empty(const Empty & rhs) {};
	~Empty() {};
	Empty & operator=(const Empty & rhs) {};
};
```



### 06 若不想使用编译器自动生成的函数，就该明确拒绝

```
1. 为驳回编译器自动（暗自）提供的函数，可将对应的成员函数声明为 private 并且不予实现。

2. 可以基类构造和析构私有函数，派生类私有继承且不声明构造和析构，这样派生类构造时就会调用基类，但是不被允许
class Uncopyable 
{
protected:
	Uncopyable() {};
	~Uncopyable() {};
private:
	Uncopyable(const Uncopyable &);
	Uncopyable & operator=(const Uncopyable &);
};

class HomeForSale : private Uncopyable {};
```



### 07 为多态基类声明 virtual 析构函数

```
1. 带多态性质的基类应该声明一个 virtual 析构函数
	如果 class 带有任何 virtual 函数，它就应该拥有一个 virtual 析构函数
	原因：派生类其中一个 delete 会删除自己和基类，导致其他派生类的对象不会被释放
	
2. 类的设计目的如果不是作为基类使用，或不是具有多态性质，就不该声明 virtual 析构函数
```



### 08 别让异常逃离析构函数

```
1. 析构函数绝对不要吐出异常
	如果析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后处理他们（不传播）或结束程序

2. 如果需要没个操作函数在运行期间抛出异常作出反应
	class 应该提供一个普通函数执行该操作（而非在析构函数中）
```



### 09 绝不在构造函数和析构函数中调用 virtual 函数

```
1. 在构造和析构函数期间不要调用 virtual 函数，因为这类调用不会下降至派生类。
	原因：基类会在派生类之前构造，而基类构造期间， virtual 函数不是 virtual 函数
```



### 10 令 operator= 返回一个 reference to *this

为了实现链式编程（大家都这样做）

```c++
class Widget 
{
public:
	Widget & operator=(int rhs)
	{
		return *this;
	}
};
```



### 11 在 operator= 中处理自我赋值

  初始数据

```c++
class Bitmap { };

class Widget
{
private:
	Bitmap * pb;
};
```

自我赋值

```
1. 直接删除，完全不安全
	Widget & Widget::operator=(const Widget & rhs)
	{
		delete pb;
		pb = new Bitmap(*rhs.pb);
		return *this;
	}

2. 证同测试，不安全，如果 new 异常导致指向指针异常
	Widget & Widget::operator=(const Widget & rhs)
	{
		if (this == rhs) { return *this; }
		delete pb;
		pb = new Bitmap(*rhs.pb);
		return *this;
	}

3. 安全性，记住原来指针
	Widget & Widget::operator=(const Widget & rhs)
	{
		Bitmap * pOrig = pb;
		pb = new Bitmap(*rhs.pb);
		delete pOrig;
		return *this;
	}
```



```
1. 确保当对象自我赋值是 operator= 有良好的行为
	比较“来源对象”和“目标对象”地址，语句顺序，以及 copy-and-swap
	
2. 确保任何函数如果操作一个以上的对象，其中多个对象是同一个对象时，其行为正确
```



### 12 复制对象时勿忘其每一个部分

```
1. 拷贝赋值函数应该确保“对象类的所有成员变量”及“所有基类部分”
	复制所有的 local 成员函数
	调用所有基类内适当的拷贝函数

2. 不要尝试某个拷贝函数实现另一个拷贝函数
	应该将共同机能放进第三个函数中，并由两个拷贝函数共同调用
```



## 3 资源管理

### 13  以对象管理资源

RAII

+ 构造时获取对应的资源，在对象生命期内控制对资源的访问，使之始终保持有效
+ 析构的时候，释放构造时获取的资源。

```
1. 为防止资源泄露，请使用 RAII （资源获取就是初始化）对象
	在构造函数中获取资源并在析构函数中释放资源

2. 两个常被使用的 RAII classes 分别是 tr1::shared_ptr 和 auto_ptr
	前者是较佳选择，因为其 copy 行为比较直观
	auto_ptr 复制动作会使它指向 null
```



### 14 在资源管理类中小心 copying 行为

```
1. 复制 RAII 对象 必须一并复制他所管理的资源
	所以资源的 copying 行为决定 RAII 对象的 copying 行为

2. 常见 RAII classes copying 行为
	抑制 copying
	施以引用记数法
```



### 15 在资源管理类中提供对原始资源的访问

```
1. APIs 往往要求访问原始资源（raw resources）
	所以每一个 RAII class 应该提供一个“取得其所管理资源”的办法
	例如智能指针能够获取原始指针的 get() 方法
2. 对原始资源的访问可能经由显示转换或隐式转换
	一般显示转换更安全，但是隐式转换对客户比较方便
	class Font
	{
	public:
		explicit Font(FontHandle fh) : f(fh) {};
		~Font() { releaseFont(f); }
		FontHandle get() { return f; };				// 显示转换 f.get()
		operator FontHandle() const { return f; };	// 隐式转换 f
	private:
		FontHandle f;
	}
```



### 16 成对使用 new 和 delete 时要采取相同的形式

```
1. 如果在 new 表达式中使用 []，必须在相应的 delete 表达式中也使用 []
	如果是指针，可能重定义，也要使用 []
	typedef std::string AddressLines[4];
	std::string * pal = new AddressLines;
	delete [] pal;
```



### 17 以独立语句将 newed 对象置入智能指针

```
1. 以独立语句将 newed 对象置入智能指针
	如果不这样做，一旦抛出异常，有可能导致难以察觉的资源泄露
```



## 4 设计与声明

### 18 让接口容易被正确使用，不易被误用

```c++
struct Day 
{
    int val;
    explicit Day(int d) : val(d) {}
}
struct Month 
{
    int val;
    explicit Month(int m) : val(m) {}
}	
struct Year 
{
    int val;
    explicit Year(int y) : val(y) {}
}
class Date
{
    Date(const Month & m, const Day & d, const Year & y);
}
Date date(Month(8), Day(1), Year(2021));	// 这样能够规定年月日，还能继续定制月份
```

```
1. 好的接口容易被正确使用，不易被误用。你应该在所有接口中努力达成这个性质
	
2. “促进正确使用”的办法包括一致性，以及与内置类型的行为兼容
	例如 STL 中每个模板返回大小接口都是 size()

3. “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值以及消除客户的资源管理责任

4. tr1::shared_ptr 支持定制型删除器（custom deleter）。这可防止 DLL 问题，可被用来自动解除互斥锁（mutexes）等
```



### 19 设计 class 犹如设计 type

好的 types 有自然的语法，直观的语义，以及一个或多个高效实现品。

```
1. 新types对象应该如何被创建和销毁?
	涉及到构造函数,析构函数,内存分配和释放函数(operator new,operator new[],operator delete,operator delete[])的设计

2. 对象的初始化和对象的赋值该有怎样的差别?
	涉及到构造函数和赋值操作符的行为以及它们的差异

3. 新type的对象如果如果被pass-by-value(以值传递),意味着什么?

4. 什么是type的合法值?
	对于class的成员变量而言,可能只有某些数据集是有效的,此时某些成员函数(特别是构造函数,赋值操作符和"setter"函数)必须进行的错误和检查工作,它也影响函数抛出的异常,以及(极少使用的)函数异常明细列.

5. 你的type需要配合某个继承图系(inheritance graph)吗?
	如果设计的type继承自某些类,就会收到哪些类的"函数是virtual或non-virtual"的影响;
	根据是否设计的type是否被继承,判断所声明的函数(尤其是析构函数)是否为虚.

6. 你的新types需要什么样的转换?
	如果需要隐式转换,可以重载类型转换函数或允许non-explict-one-arguement(非explict单实参)构造函数.
	如果只允许显示转换,就专门写出负责执行转换的函数,且禁止类型转换操作符和non-explict-one-arguement(非explict单实参)构造函数

7. 什么样的操作符和函数对此新type而言是合理的?

8. 什么样的标准函数应该驳回?
	声明为private或只声明不定义.(具体见条款6)

9. 谁该取用新type的成员?
	决定哪些成员为public,哪些为protect,哪些为private,那些类和函数是friends,以及将它们嵌套于另一个之内是否合理.

10. 什么是新type的未声明接口?
	明确它对效率,异常安全性(见条款29),以及资源运用(例如多任务锁定和动态内存)提供何种保证.

11. 你的新type有多么一般化?
	判断是否直接定义一个新的class template.

12. 你真的需要一个新type吗?
	如果只是为已有类添加新功能,说不定单纯定义一个或多个non-member函数或template即可.
```



### 20 宁以 pass-by-reference-to-const 替换 pass-by-value

```
1.尽量以 pass-by-reference-to-const 替换 pass-by-value。前者通常比较高效，并可避免切割问题。
	1.1 高效
	自定义对象，值传递往往需要拷贝构造调用和析构函数调用创建一个新对象，内部如果还有还需继续拷贝和析构
	引用传递，底层是指针实现，相当于调用自身，不会创建新对象
    1.2 切割问题
    值传递会导致特化性质被切割掉，只能调用值传递对象的方法（拷贝问题）
    引用传递可以根据传递参数对象调用特化性质的方法
    
2. 以上规则并不适用于内置类型，以及 STL 的迭代器和函数对象。
	对他们而言 值传递 往往比较合适
```



### 21 必须返回新对象时，别妄想返回其 reference

```
1. 一个“必须返回新对象”的正确写法是：就让那个函数返回一个新对象
	inline const Rational operator*(const Rational & lhs, const Rational & rhs)
	{
		return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
	}
	
2. 绝不要返回 pointer 和 reference 指向一个 local stack 对象
	或返回 reference 指向一个 heap-allocated 对象
	或返回 pointer 和 reference 指向一个 local static 对象
```



### 22 将成员变量声明为 private

```
1. 切记将成员变量声明为 private。
	这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，
	并提供 class 作者充分的弹性
	
2. protected 并不比 public 更具封装性
```



### 23 宁以 non-mumber、non-friend 替换 member 函数

```
1. 宁以 non-mumber、non-friend 替换 member 函数
	这样做可以增加封装性、包裹弹性和机能扩充性
	非成员函数能够更加直观反应调用函数，编译相依度更低，扩展更容易
```

```c++
class WebBrowser
{
    public:
    void clearCache();
    void clearHistory();
    void removeCookies();

    // 1. 成员函数清除所有
    void clearEverything();
}
// 2. 非成员函数清除所有，调用封装好的模块函数
void clearBrowser(WebBrowser & wb)
{
    wb->clearCache();
    wb->clearHistory();
    wb->removeCookies();
}
```



### 24 若所有参数皆需类型转换，请为此采用 non-member 函数

```
1. 如果需要为某个函数的所有参数（包括被this指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是non-member函数
	第一种 
	res = Rational(1, 2) * 2;	// 可以，2 隐式转换（有默认有参构造）
	res = 2 * Rational(1, 2);	// 不可以，前面是调用方，不能隐式转换
	所以为了通用采用非成员函数
```



```c++
// 成员函数
const Rational operator*(const Rational & rhs)
{
	return Rational(this.n * rhs.n, this.d * rhs.d);
}

// 非成员函数
const Rational operator*(const Rational & lhs, const Rational & rhs)
{
	return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
}
```



### 25 考虑写一个不抛出异常的 swap 函数

```
1. 当 std::swap 对你的类型效率不高时，提供一个 swap 成员函数，并确定这个函数不抛出异常

2. 如果你提供一个 member swap，也该提供一个 non-member swap用来调用前者
	对于 classes 也请特化 std::swap
	
3. 调用 swap 时应针对 std::swap 使用 using 声明式，然后调用 swap 并且不带任何“命名空间修饰”

4. 为“用户定义类型”进行 std templates 全特化，但千万不要尝试在 std 内加入某些对 std 而言全新的东西
```

 

## 5 实现

### 26 尽可能延后变量定义式的出现时间

```
1. 尽可能延后变量定义式的出现时间
	这样可以增加程序的清晰度并改善程序效率

2. 循环外还是循环内定义变量
	循环外：1 个构造，1 个析构，n 个赋值
	循环内：n 个构造，n 个析构
```



### 27 尽量少做转型动作

```
1. 分类
	const_cast			用来将对象的常量性转除
	dynamic_cast		安全向下转型
	reinterpret_cast	低级转型，不可移植，指针转基本类型
	static_cast			强制隐式转换

2. 转型样式
	(T)expression	// C 旧式
	T(expression)	// C++ 旧式
	static_cast<T>(expression)	// C++ 新式
	
3. 如果可以，尽量避免转型操作
	特别是注重效率，需要避免 dynamic_cast

4. 如果转型必要，可以将代码隐藏于某个函数后，客户可以调用而不需要自己转型
	
5. 宁可使用C++新式转型，不要使用旧式转型
	容易分辨，且知道作用类型
```



### 28 避免返回 handles 指向对象内部成分

```
1. 避免返回 handles （包括引用，指针，迭代器）指向对象内部成分
	可以增加封装性，帮助 const 成员函数行为像个 const，降低“虚吊”（返回一个临时指针，临时指针被销毁，导致指向错误）的发生
	
	const Point & upperLeft() const { return pData-> ulhc; };
```



### 29 为异常安全努力是值得的

```
1. 异常安全函数即使发生异常也不会泄露资源或允许数据结构破坏
	基本型：没有任何对象或数据结构因此破坏，保持有效
	强烈型：程序状态不会改变，成功完全执行，失败，返回调用函数之前的状态
	不抛异常型：承诺不抛出异常

2. 强烈保证往往以 copy-and-swap 实现，但并非对所有函数都可实现或有现实意义
	就是先拷贝一个副本，再在副本上进行操作，失败原数据不做修改，成功将副本修改数据在一个不抛出异常的函数中进行置换

3. 函数提供的异常安全保证最高只等于调用各个函数的最弱者
```



### 30 透彻了解 inlining 的里里外外

```
1. inline 不是强制命令，只是一个申请，分为显式和隐喻
	隐喻：将函数声明和实现都放在 class 中
	class Person
	{
		public:
			int age() const { return _Age; };
		private:
			int _Age;
	}
	显示：在函数前面加上 inline 关键字
	inline MAX(int a, int b) { return a > b ? a : b; };
	
2. 将大多数 inlining 限制在小型、被频繁调用的函数上
	日后调式和二进制升级更容易，减少代码膨胀问题，使程序速度提升最大化
	
3. 不要只因为 function templates 出现在头文件，就将它们声明为 inline
```



### 31 将文件间的编译依赖关系将至最低

```c++
class Person{
public:
    // 外部接口
    Person(const string& name);
    string name() const;
private:
    // 内部实现
    string _name;
    Date _birthday;
};
```

`<string>`中定义了`string`类，`date.h`中定义了`Date`类。这些`include`在编译前都是要拷贝进来的！这使得`Person`与这些头文件产生了编译依赖。 只要这些头文件（以及它们依赖的文件）中的类定义发生改动，`Person`类便需要重新编译。

```
1. 支持"编译依存性最小化"的一般构想是：相依于声明式，不要相依于定义式。
	基于此构想的两个手段是Handle classes和Interface classes

2. 程序库头文件应该以“完全且仅有声明式”(full and declaration-only forms)的形式存在。这种做法不论是否涉及template都适用。
```

```c++
// 句柄类
class PersonImpl; // 实现类的前置声明

class Person
{
public:
    Person(std::string & name);
    std::string name() const;
private:
    shared_ptr<PersonImpl> pImpl;
};

Person::Person(std::string & name): pImpl(new PersonImpl(name)) {}
std::string Person::name() { return pImpl->name(); }
```

```c++
// 接口类
class Person
{
public:
    virtual ~Person();
    virtual std::string name() const = 0;
    virtual std::string birthday() const = 0;
    virtual std::string address() const = 0;
};

// 声明
class Person
{
public:
    static boost::shared_ptr<Person> create(std::string & name);
};

boost::shared_ptr<Person> Person::create(std::string & name)
{
    return shared_ptr<Person>(new RealPerson(name));
}

// 使用
boost::shared_ptr<Person> pp(Person::create("alice"));
```



## 6 继承与面向对对象设计

### 32  确定你的 public 继承塑模出 is-a 关系

```
1. “public继承”意味着 is-a
	适用于基类的每一个事件也适用于派生类
	因为每一个派生类对象也是一个基类对象
```



### 33 避免遮掩继承而来的名称

```
1. 派生类中的名称会遮掩基类中的名称（类型不一样也会）
	int x;
	void someFunc()
	{
		double x;
		std::cin >> x;
	}

2. 为了让被遮掩的名称重见天日，可以使用 using 声明式或转交函数（forwarding function）
```

```c++
// using 声明式
class Base
{
private:
	int x;
public:
	virtual void mf1() = 0;
	virtual void mf1(int);
	virtual void mf2();
	virtual void mf3();
	virtual void mf3(double);
}

class Derived : public Base
{
public:
	using Base::mf1;	// 让基类中 mf1 和 mf3 所有东西在基类中可见
	using Base::mf3;
	virtual void mf1();
	void mf3();
}
```



```c++
// 转交函数
class Base
{
private:
	int x;
public:
	virtual void mf1() = 0;
	virtual void mf1(int);
}

class Derived : public Base
{
public:
	virtual void mf1() { Base::mf1(); }
}
```



### 34 区分接口继承和实现继承

缺省实现：在函数的参数列表设置的默认值

```
继承分类
(1)只继承接口
对应的成员函数的写法为：pure virtual。
特点：此类继承派生类必须自己写实现。
注意：pure virtual 函数是可以写定义的，但是只能通过类名调用。

(2)同时继承接口和实现，且继承而来的实现能够被覆写
对应的成员函数的写法：virtual 。
特点：此类继承派生类可以缺省实现，缺省的话，就会继承基类的实现。也可以自己手动写实现，这样相当于是覆盖了基类的实现。

(3)同时继承接口和实现，且继承而来的实现不能够被覆写
对应的成员函数的写法：non-virtual。
特点：此类继承，派生类必须继承缺省和实现，且不能够修改。
```

```
1. 接口继承和实现继承不同。在public继承之下，派生类总是继承基类的接口。

2. pure virtual函数只具体指定接口继承

3. impure virtual（普通虚函数）函数具体指定接口继承及缺省实现继承

4. non-virtual(普通的非虚函数) 函数具体指定接口继承以及强制性实现继承
```



### 35 考虑 virtual 函数以外的其他选择

```
1. virtual函数的替代方案包括NVI(non virtual interface)手法以及Strategy设计模式的多种形式，NVI手法自身是一个特殊形式的Template Method设计模式；

2. 将机能从成员函数转移到class外部函数，带来的一个缺点是：非成员函数无法访问class 的non-public成员；

3. trl::function对象的行为就像一般函数指针。这样的对象可接纳“与给定之目标签名式兼容”的所有可调用物(callable entities)。
```



### 36 绝不重新定义继承而来的 non-virtual 函数

```
重新定义请声明为 virtual 函数
```



### 37 绝不重新定义继承而来的 缺省参数值

```
1. 绝不重新定义继承而来的 缺省参数值
	因为缺省参数值都是静态绑定的，而 virtual 函数（应该覆写的东西）是动态绑定的
	就是调用一个定义于派生类的 virtual 函数，却是使用基类为它指定的缺省参数值
```

```c++
class Shape 
{
public:
    enum ShapeColor { Red, Green, Blue };
    virtual void draw(ShapeColor color = Red) const = 0;
};
 
class Rectangle : public Shape 
{
public:
    virtual void draw(ShapeColor color = Green) const;
};
 
class Circle : public Shape 
{
public:
    virtual void draw(ShapeColor color) const;
}
```



### 38 通过复合塑模出 has-a 或“根据某物实现出”

```
1. 复合（composition）的意义和 public 继承完全不同
	复合（has-a）：有一个
	public继承（is-a）：是一种
	
2. 在应用域复合意味着（has-a），在实现域复合意味着（根据某物实现出）
```



### 39 明智而审慎的使用 private 继承

```
1. 私有继承意味着根据某物实现出。它通常比复合的级别低
	当继承类需要访问基类中的保护成员（protected），或需要重新定义继承而来的虚函数时，这么设计是合理的。
	1）private 继承不会自动将派生类转换为基类对象，所以调用基类方法失败
	2）private 继承会将所有成员都变成 private
	
2. 和复合不同，私有继承可以造成空白基类最优化（empty Base optimization, EBO），单一继承有效
	这对致力于对象尺寸最小化的程序库开发者而言，可能很重要。
	// sizeof(HoldsAnInt) > sizeof(int)
	class Empty() {}
	class HoldsAnInt
	{
	private:
		int x;
		Empty e;
	}
	
	// sizeof(HoldsAnInt) == sizeof(int)
	class HoldsAnInt : private Empty
	{
	private:
		int x;
	}
```



### 40 明智而审慎的使用多重继承

```
1. 多重继承比单一继承更复杂。它可能导致新的歧义性，以及对virtual继承的需要。
	歧义性：属性或方法名字冲突

2. virtual继承会增加大小、速度、初始化（及赋值）复杂度等等成本。
	如果virtual base classes不带任何数据，将是最具实用价值的情况。

3. 多重继承的确有正当用途。其中一个情节涉及“public继承某个Interface class”和“private继承某个协助实现的class”的两两组合。
```



## 7 模板与泛型编程

### 41 了解隐式接口和编译期多态

```
1. classes和templates都支持接口和多态。

2. 对classes而言接口是显示的，以函数签名（函数名称、参数类型、返回类型）中心。
	多态则是通过virtual函数发生于运行期。
class Widget
{ 
public: 
    Widget(); 
    virtual ~Widget(); 
    virtual std::size_t size() const; 
    virtual void normalize(); 
    virtual swap(Widget& other); 
}; 

void doProcessing(Widget& w) 
{ 
    if (w.size() > 10 && w != someNastyWidget)
    { 
        Widget temp(w); 
        temp.normalize(); 
        temp.swap(w); 
    } 
}

3. 对template参数而言，接口是隐式的，奠基于有效表达式。
	多态则是template具现化和函数重载解析发生于编译期。
template<typename T>
void doProcess(T & w) 
{
	if(w.size() > 10 && w != someNastyWidget)
	{}
}
```



### 42 了解 typename 的双重意义

```
1. 声明template参数时，前缀关键字class和typename可以互换

2. 使用关键字typename标识嵌套从属类型名称
	template内出现的名称相依于某个 template 参数，称为从属名称
	如果从属名称在 class 内呈嵌套状，称为嵌套从属名称
	但不得在base class list（基类列）或member initialization list（成员初值列）内以它作为base class修饰符

template<typename C>
void print2nd(const C & container)
{
    if(container.size()>=2)
    {
    	typename C::const_iterator iter(container.begin());
    }
}

```



### 43 学习处理模块化基类内的名称

```
1. 可在 derived class templates 内通过 “this->” 指向 base class template 内的成员名称
	或由一个明白写出的 “base class 资格修饰符” 完成
```

```c++
// 在基类函数调用之前加上“this->”
template<typename Company>
class LoggingMsgSender:public MsgSender<Company>
{
public:
	void sendClearMsg(const MsgInfo& info)
	{
		this->sendClear(info);	// 成立，假设sendClear 将被继承 
	}
};

// 使用using声明
template<typename Company>
class LoggingMsgSender:public MsgSender<Company>
{
public:
	using MsgSender<Company>::sendClear;	// 告诉编译器，请它假设sendClear位于base class内
	void sendClearMsg(const MsgInfo& info)
	{
		sendClear(info);	// 成立，假设sendClear将被继承下来
	}
}

// 明确指出被调用的函数位于基类内
template<typename Company>
class LoggingMsgSender:public MsgSender<Company>
{
public:
	void sendClearMsg(const MsgInfo& info)
	{
		Msgsender<Company>::sendClear(info);	// 成立，假设sendClear将被继承下来
	}
};
```



### 44 将与参数无关的代码抽离 template

```
1. Template生成多个classes与多个函数，所以任何template代码都不该与某个造成膨胀的template参数产生相依关系

2. 因非类型模板参数而造成的代码膨胀，往往可以消除，做法是以函数参数或者class成员变量替换template参数

3. 因类型而造成的代码膨胀，也可以降低，做法是让带有完全相同二进制表述的具现类型共享实现码
```

```c++
// 典型例子
template <typename T, std::size_t n>
class Matrix {
  public:
    void invert();
};
// 会具现两份非常相似的代码，除了一个参数5，一个参数10
Matrix<double, 5> m1;
Matrix<double, 10> m2;


// 改进一使用带参数值的函数
template <typename T>
class MatrixBase 
{
protected:            // protected 保证只有本类/子类本身可以调用
    void invert(std::size_t n);
}
template<typename T, std::size_t n>
class Matrix : private MatrixBase<T> 
{ 
// private继承，derived 和base不是is-a关系，base只是帮助实现derived 
private:
    using MatrixBase<T>::invert;       // derived class 会掩盖template base class的函数             
public:
    inline void invert() { this->invert(n);}
}


// 改进二为矩阵添加数据指针
template <typename T>
class MatrixBase 
{
protected:
    MatrixBase(T* data, n) : data_(data), n_(n);
    void setDataptr(T* data) { data_ = data; }
private:
    T& data_;
    std::size_t n_;
};
template <typename T, std::size_t n>
class Matrix : private MatrixBase<T>
{
public:
     // 先初始化base class,再初始化derived class member
    Matrix() : MatrixBase<T>(n, nullptr), sptr_(new T[n*n]) 
    {         
    	this->setDataptr(sptr_.get());	// 再为base class member 赋值
    }
private:
   boost:scoped_array<T> sptr_;
};
```



### 45 运用成员函数模板接受所有兼容类型

```
1. 请使用成员函数模板（member function templates）生成“可以接受所有兼容类型”的函数

2. 若你声明member templates用于"泛化拷贝构造"或"泛化赋值操作"，你还需要声明正常的拷贝构造函数和拷贝赋值操作符
```



### 46 需要类型转换时请为模板类定义非成员函数

```
1. 当我们编写一个class template时候，它提供“与此template相关的”函数支持“所有参数之隐式类型转换”时，请将那些函数定义为“class template内部的friend函数”
	原因：混合式运算（mixed-mod）一般需要类型转换
```

```c++
// 1. 内部直接
template <typename T>
class Rational
{
public:
    friend const Rational operator*(const Rational & lhs, const Rational & rhs)
    {
        return Ratioanl(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
    }
};

// 2. 外部辅助函数
template <typename T>
const Rational<T> doMultiply(const Rational<T> & lhs,const Rational<T> & rhs);

template <typename T>
class Rational
{
public:
    friend Rational<T> operator*(const Rational<T> & lhs, const Rational<T> & rhs)
    {
        return doMultiply(lhs, rhs);
    }
};
```



### 47 请使用 traits classes 表现类型信息

```
1. 建立一组重载函数(身份像劳工)或函数模板(例如doAdvance),彼此之间的差异仅在于各自的traits参数
	令每个函数实现码与其接受之traits相应和

2. 建立一个控制函数(身份像工头)或函数模板(例如advance),它调用上述"劳工函数并传递traits classes所提供的信息"
```

```c++
// C++ 内置迭代器
struct input_iterator_tag{};
struct output_iterator_tag{};
struct forward_iterator_tag:public input_iterator_tag{};
struct bidirectional_iterator_tag:public forward_iterator_tag{};
struct random_access_iterator_tag:public bidirectional_iterator_tag{};
```



### 48 认识 template 元编程

```
1. 模板元编程（Template metaprogramming,TMP）可将工作由运行期移往编译期，因而得以实现早期错误侦测和更高的执行效率

2. TMP可被用来生成“基于政策选择组合(based on combinations of policy choices)”的客户定制代码
	也可用来避免生成对某些特殊类型并不适合的代码
```



## 8 定制 new 和 delete

### 49 了解 new-handler 的行为

new抛出一场以反映一个未获满足的内存需求之前,会先调用一个用户指定的错误处理函数.这个就是new-handler.

```
1. 作为set-new-handler函数中的参数的new-handler允许用户自定义，它在内存分配不足时被调用

2. Nothrow new太局限，它只适合于内存分配，对于后续的构造函数调用还是可能抛出异常
```



### 50 了解 new 和 delete 的合理替换实际

```
1. 用来检测运用上的错误
	如果将“new所得内存”delete掉却不幸失败，会导致内存泄漏（memory leaks）
	如果在“new所得内存”身上多次delete则会导致不确定行为
	如果operator new持有一串动态分配所得地址，而operator delete将地址从中移走，倒是很容易检测出上述错误用法
	
2. 为了强化效能
	它们必须接纳各种分配形态，范围从程序存活期间的少量区块动态分配，到大数量短命对象的持续分配和归还
	它们必须考虑破碎问题，这最终会导致程序无法满足大区块内存要求，即使彼时有总量足够但分散为许多小区块的自由内存

3. 为了收集使用上的统计数据
	在一头栽进定制型news和定制型deletes之前，理当收集你的软件如何使用其动态内存
	
4. 为了增加分配和归还的速度
	泛用型分配器往往（虽然并不总是）比定制型分配器慢，特别是当定制型分配器专门针对某特定类型之对象而设计时

5. 为了降低缺省内存管理器带来的空间额外开销
	泛用型内存管理器往往（虽然并非总是）不只比定制型慢，它们往往还使用更多内存，那是因为它们常常在每一个分配区块身上招引某些额外开销。针对小型对象而开发的分配器本质上消除了这样的额外开销。

6. 为了弥补缺省分配器中的非最佳齐位
	在x86体系结构上doubles的访问最是快速——如果它们都是8-byte齐位
	但是编译器自带的operator news并不保证对动态分配而得的doubles采取8-byte齐位
	这种情况下，将缺省的operator new替换为一个8-byte齐位保证版，可导致程序效率大幅提升。

7. 为了将相关对象成簇集中
	如果你知道特定之某个数据结构往往被一起使用，而你又希望在处理这些数据时将“内存页错误”的频率降至最低，那么为此数据结构创建另一个heap就有意义，这么一来它们就可以被成簇集中在尽可能少的内存页上
	new和delete的“placement版本”（见条款52）有可能完成这样的集簇行为

8. 为了获得非传统的行为
	有时候你会希望operators new和delete做编译器附带版没做的某些事情
	例如你可能会希望分配和归还共享内存内的区块，但唯一能够管理该内存的只有C API函数，那么写下一个定制版new和delete，你便得以为C API穿上一件C++外套。你也可以写一个自定的operator delete，在其中将所有归还的内存内容覆盖为0，籍此增加应用程序的数据安全性。
```

有许多理由需要写个自定的new和delete，包括改善效能、对heap运用错误进行调试、收集heap使用信息。



### 51 编写 new 和 delete 时需固守常规

```
1. operator new应该内含一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就该调用new-handler
	它也应该有能力处理0 bytes申请（视为1-byte请求）

2. Class operator delete应该收到null指针时不做任何事。Class专属版本则还应该处理“比正确大小更大的（错误）申请”
```



### 52 写了 placement new 也要写 placement delete

```
1. 当你写一个placement operator new，请确定也写出了对应的placement operator delete
	如果没有这样做，你的程序可能会发生隐微而时断时续的内存泄漏

2. 当你声明placement new和placement delete，请确定不要无意识地掩盖了它们的正常版本
```

 ```c++
 // inherit std forms
 class Widget: public StandardNewDeleteForms 
 {
 public:
    using StandardNewDeleteForms::operator new;         
    using StandardNewDeleteForms::operator delete;   // 让这些形式可见    
  
    static void* operator new(std::size_t size, std::ostream& log) throw(std::bad_alloc);   // 自定义 placement new
    static void operator delete(void *pMemory, std::ostream& logStream) throw();            // 对应的 placement delete
 };
 ```



## 9 杂项讨论

### 53 不要轻易忽视编译器的警告

```
1. 严肃对待编译器发出的警告
	努力在你的编译器的最高警告级别下争取“无任何警告”荣誉
	
2. 不要过度依赖编译器的报警能力
	因为不同的编译器对待事情的态度并不相同，一旦移植到另一个编译器上，你原本依赖的警告信息有可能消失
```



### 54 让自己熟悉包括 TR1 在内的标准程序库

```
1. C++标准程序库的主要机能由STL、iostreams、locales组成。并包括C99标准程序库

2. TR1添加了智能指针、一般化函数指针、hash-based容器、正则表达式，以及另外10个组件的支持

3. TR1自身只是一个规范。为获得tr1提供的好处，你需要一份实物。一个好的实物来源是boost
```



### 55 让自己熟悉 Boost

```
1. 网址
	http://boost.org

2. Boost是一个社群，也是一个网站
	致力于免费的、源码开放、同僚复审的C++程序库开发。Boost在C++标准化过程中扮演深具影响力的角色

3. Boost提供许多TR1组件实现品，以及其他许多程序库
```



## 附录

### 1 标准

| 发布时间 |        文档        |                       通称                       |     备注      |
| :------: | :----------------: | :----------------------------------------------: | :-----------: |
|   2020   | ISO/IEC 14882:2020 | [C++20](https://zh.wikipedia.org/wiki/C%2B%2B20) |               |
|   2017   | ISO/IEC 14882:2017 | [C++17](https://zh.wikipedia.org/wiki/C%2B%2B17) | 第五个C++标准 |
|   2014   | ISO/IEC 14882:2014 | [C++14](https://zh.wikipedia.org/wiki/C%2B%2B14) | 第四个C++标准 |
|   2011   | ISO/IEC 14882:2011 | [C++11](https://zh.wikipedia.org/wiki/C%2B%2B11) | 第三个C++标准 |
|   2003   | ISO/IEC 14882:2003 |                      C++03                       | 第二个C++标准 |
|   1998   | ISO/IEC 14882:1998 |                      C++98                       | 第一个C++标准 |

