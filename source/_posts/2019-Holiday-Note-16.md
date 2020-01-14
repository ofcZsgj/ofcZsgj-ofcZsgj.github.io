---
title: Holiday Note_16
date: 2019-07-28 21:25:16
categories: C++
tags: [Note,C++]
---

# Holiday Note_16

终于接触到了C++的**模板机制**。
> 《C++ Primer Plus》： 模板提供参数化(parameterized)类型，即能够将类型名作为参数传递给接收方来建立类或函数

<!-- more -->

## 函数模板

### 定义及写法

简单来说，函数模板便是让函数的参数类型不再固定，而是让数据类型也参数化，即可以根据不同的数据类型进行转化从而达到函数重用的功能。

> C++提供了函数模板(function template.)所谓函数模板，实际上是建立一个**通用函数**，其**函数类型和形参类型不具体制定**，用一个虚拟的类型来代表。这个通用函数就成为函数模板。凡是函数体相同的函数都可以用这个模板代替，不必定义多个函数，只需在模板中定义一次即可。在**调用函数时系统会根据实参的类型来取代模板中的虚拟类型**，从而实现不同函数的功能。

> 	模板把函数或类要处理的数据类型参数化，表现为**参数的多态性**，成为**类属**。
>	模板用于表达**逻辑结构相同**，但具体**数据元素类型不同**的数据对象的通用行为。

---

一个交换函数，在不使用函数模板前，需要对int型数据写一个swap函数，对double型数据写一个swap函数...而代码除了数据类型的不同，核心的代码几乎一致。

而使用函数模板，定义一个类型T(**定义的这个类型T只对其后的唯一一个类或函数生效**)，将数据类型参数化，可以大大的增强函数的重用性。
~~~
template<class T>
void MySwap(T& a,T& b){
	T temp = a;
	a = b;
	b = temp;
}
~~~
调用时，函数模板可以自动推导参数的类型
~~~
int a = 10;
int b = 20;
MySwap(a,b);
~~~
也可以显式指定类型
~~~
char c1 = 'a';
char c2 = 'b';
MySwap<char>(c1, c2);
~~~

---

### 与普通函数的区别
-	函数模板**不允许自动类型转化**
-	普通函数能够自动进行类型转化

即定义一个int类型的变量a和一个char类型的b，使用swap函数时，若使用普通函数的情况下，参数为int时，char类型的b会转换为ASCLL码的int数值进行比较；而模板函数必须严格匹配类型。

---

### 函数模板和普通函数在一起调用规则
-	c++编译器**优先考虑普通函数**
-	可以通过**空模板实参列表**的语法**限定编译器只能通过模板匹配**(如swap<>(a, b))
-	函数模板**可以**像普通函数那样可以被**重载**
-	如果函数模板可以产生一个**更好的匹配**(即在普通函数需要进行类型转换时)，那么选择模板

---

### 函数模板机制

-	编译器并不是把函数模板处理成能够处理任何类型的函数
-	函数模板通过具体类型产生不同的函数
-	**编译器会对函数模板进行两次编译**，在**声明**的地方对模板代码本身进行编译，在**调用**的地方对参数替换后的代码进行编译。

 **但是**，编写的模板函数无法处理自定义类型，因此需要通过对模板函数进行**重载**，提供**具体化模板**。

比如这样一个自定义的Person类，编译器无法处理，需要对模板进行**第三代具体化实现数据类型的匹配**
~~~
class Person {
public:
	int m_Age;
	string m_Name;
};
~~~
函数模板
~~~
template<class T>
bool myCompare(T& a, T& b) {
	if (a == b) {
		return true;
	}
	else return false;
}
~~~
具体化模板写法为**template<> 返回值 函数名<具体类型>(参数类型)**
~~~
template<> bool myCompare<Person>(Person& a, Person& b) {
	if (a.m_Age == b.m_Age && a.m_Name == b.m_Name) {
		return true;
	}
	else return false;
}
~~~

---

## 类模板

### 写法

和函数模板类似，类模板是让类中的数据类型参数化

类模板中的**成员函数** 一开始不会创建出来，而是在**运行时才去创建**
~~~
template<class NameType, class AgeType>
class Person
{
public:
	NameType mName;
	AgeType mAge;
};
~~~
但是要注意的是：**类模板不能进行类型自动推导**，需要<>提供参数列表
~~~
void test01()
{
	Person<string, int>P1("AAA", 10);
	P1.showPerson();
}
~~~

类模板类内定义，类外实现时，需要在实现前加template等，作用域前还要加参数列表。
~~~
//构造函数
template<class T1, class T2>
Person<T1, T2>::Person(T1 name, T2 age){
	this->mName = name;
	this->mAge = age;
}
~~~

---

### 子类继承类模板
父类类模板
~~~
template<class T>
class Base
{
	T m;
};
~~~
继承类模板的时候，必须要确定基类的大小。即在基类名后加<类型>
~~~
template<class T >
class Child2 : public Base<double>
{
public:
	T mParam;
};
~~~

---

###	类模板做函数参数的三种方式	
-	显示指定类型
-	参数模板化
-	整体模板化

针对Person类中的showPerson方法，以下为类模板做参数的三种方式

指定传入类型(即普通函数，编译器优先调用)
~~~
void doWork(Person<string, int>& p) {
	p.showPerson();
}
~~~
参数模板化
~~~

template<class T1,class T2>
void doWork(Person<T1,T2>& p) {
	p.showPerson();
}
~~~
整体模板化
~~~
template<class T>
void doWork(T& p) {
	p.showPerson();
}
~~~

---

### 类模板分文件编写


1. 问题
-  .h .cpp分别写声明和实现
-	但是由于类模板的成员函数运行阶段才去创建，导致包含.h头文件，不会创建函数的实现，**无法解析外部命令**
2.	解决方案（不推荐导入.cpp文件 ）
-	不要进行分文件编写，写到同一个文件中，进行声明和实现，后缀名改为**.hpp**




查看类型名称的方法
~~~
cout << typeid(T).name() << endl;
~~~

---

### 类模板与友元函数

1.	友元函数类内实现
-	friend void printPerson( Person<T1 ,T2> & p ) 
2. 友元函数类外实现（较为复杂）
-	friend void printPerson<>(Person<T1, T2> & p); //没有**<>**仅为**普通函数**因此**加上<>模板函数声明**
-	而由于这个**函数在类内**，编译器**编译时无法看到**，因此需要在**类外声明**函数并且看到这个Person类型(也是在类外提前声明)

先进行类外声明，否则文件编译时链接失败
~~~
template<class T1, class T2>class Person;
template<class T1, class T2>void PrintPerson(Person<T1, T2>& p);
~~~
类内声明需要加<>使其变成模板函数声明
~~~
template<class T1, class T2>
class Person {
public:	
	friend void PrintPerson<>(Person<T1, T2>& p);

private:
	T1 m_Age;
	T2 m_Name;
};
~~~
类外实现
~~~
template<class T1, class T2>
void PrintPerson(Person<T1, T2>& p) {
	cout << "Name: " << p.m_Age << " Age: " << p.m_Age << endl;
}
~~~

PS:颤抖的手点下了付款。
