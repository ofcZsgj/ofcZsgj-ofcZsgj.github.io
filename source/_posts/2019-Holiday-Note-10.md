---
title: Holiday Note_10
date: 2019-07-22 20:22:51
categories: C++
tags: [Note,C++]
---

# Holiday Note_10

开始学C++了。第一天，学的特别杂，主要是C++对于C的一些增强，如**全局变量检测**，**函数检测**，**类型转换检查**，**struct**，**bool**，**三目运算符**，**const**。然后学了C++中的**namespace**，**using**和**引用**的使用方法。其中 *引用(reference)* 是其中的重点，它是C++对C的重要扩充。相较于C的地址，指针传递，引用更加清晰，语法更加简单不容易出错。

<!-- more -->

## C++对C的增强
> C++之父-本贾尼·斯特劳斯特卢普(Bjarne Stroustrup)是这么说的：C++主要是为了我的朋友和我不必再使 用汇编语言、C语言或者其他现代高级语言来编程而设计的。它的主要功能是可以更方便得编写出好程序，**让每个程序员更加快乐**”。

[![CPP573f4.md.png](https://miao.su/images/2019/07/22/CPP573f4.md.png)](https://miao.su/image/TINaJ)

看得出来他很快乐👴

C++是面向对象的， 面向对象三大特性：**继承**；**封装**；**多态**；

### 全局变量检测增强
此代码在C++下编译失败,在C下编译通过
~~~
int a = 10; //赋值，当做定义
int a; //没有赋值，当做声明
~~~

### 函数检测增强

C++中所有的变量和函数都必须有类型，因此，在C中写函数可以不写返回值类型，不写参数类型，不写参数，有返回值却不写return都可以编译成功，但是C++编译均会失败。

1. 参数类型检测
2. 返回值检测
3. 传参个数检测

### 类型转换检测增强
malloc返回void* ，C中可以不用强转用想使用的指针类型接收即可，C++必须强转成接收指针类型才能编译成功；

### struct增强
- c中定义结构体变量需要加上struct关键字，C++不需要。
- c中的结构体只能定义成员变量，不能定义成员函数。C++即可以定义成员变量，也可以定义成员函数。

以下为C++可以实现的结构体定义以及使用。
~~~
struct Person {
	char name[64];
	int age;
	void ageAdd() {
		age++;
	}
};

Person p1;
p1.ageAdd();
~~~

### bool数据类型增强
C++中有bool类型，C没有。true/false。sizeof结果为1。
~~~
bool flag = true;
~~~


### 三目运算符增强
- C语言三目运算表达式**返回值为数据值**，为右值，不能赋值。
- C++语言三目运算表达式**返回值为变量本身(引用)**，为左值，可以赋值。
~~~
int a = 10, b = 20;
//下面这条语句在C中无法成功编译。C++执行完后b的值变成了100
(a > b ? a : b) = 100;
~~~

### const增强
1. **在C++中，一个const不必创建内存空间，而在C中，一个const总是需要一块内存空间。**

- C语言中const是伪常量，局部const存储在堆栈区，只是不能通过变量直接修改const只读变量的值，但是可以跳过编译器的检查，通过指针间接修改const值。

- C++中const会放入到**符号表**中，不会分配内存，而是**Key-Value**的形式，const修饰的变量为key值，变量值为value。当对其取地址时，才会分配内存，但是这个新分配出的内存只是拷贝的const修饰变量的value值，指针操作的只是这块空间，因此无法进行修改。
	

2. C语言中const默认是外部链接，C++中const默认是内部链接
	
3. const分配内存情况
	
- 对变量取地址，会分配**临时内存**
~~~
void test() {
	const int m_A = 10;//不分配内存，KEY-VALUE
	int* p = (int *)&(m_A);//分配的是临时内存
	*p = 100;

	cout << "*p = " << *p << endl;//100
	cout << "m_A = " << m_A << endl;//10
}
~~~
	
- extern关键字下的const会分配内存
	
- 用**普通变量初始化const变量**会分配内存
~~~
void test06() {
	int b = 10;
	const int m_A = b;//会分配内存
	int* p = (int*) & (m_A);

	*p = 100;

	cout << "*p = " << *p << endl;//100
	cout << "m_A = " << m_A << endl;//100
}
~~~

- **自定义数据类型**会分配内存
~~~
struct Person {
	string name = "AAA";
	int age = 10;
};

const Person p1;
//p1.name = "BBB";//虽然不能简单修改，但是通过指针取地址可以修改了
	
Person* p = (Person*) & (p1);	
p->name = "BBB";
cout << "name = " << p->name << endl;//name = BBB
~~~

4. 尽量用const代替define

- const有类型，**可进行编译器类型安全检查**。#define无类型，不可进行类型检查.
- const有**作用域**，而#define不重视作用域，默认定义处到文件结尾.如果定义在指定作用域下有效的常量，那么#define就不能用。


---

## C++中namespace和using的使用
C++的Hello World写法
~~~
//标准输入流
#include <iostream>
//使用命名空间
using namespace std;

int main01() {

	//cout：标准输出；<< 符号重载，字符拼接；endl标准结束换行
	cout << "Hello World" << endl;

	//返回正常退出
	return EXIT_SUCCESS;

	system("pause");//阻塞功能

 }
~~~
### namespace用途以及使用方法
namespace就像一个一个的房子，里边有这个房子特定的数据。使用**命名空间**是为了**更好地控制标识符的作用域**

- 解决名称冲突问题

- **必须在全局作用域下声明**

- 命名空间下可以放入 函数、变量、结构体、类…
~~~
namespace veryLongName{	
	int a = 10;
	void func(){ cout << "hello namespace" << endl; }
}
~~~

- 命名空间可以嵌套命名空间
~~~
namespace A {
	int m_A = 10;

	namespace B {
		int m_C = 30;
	}

}
~~~


- 命名空间是开放的，可以随时加入新的成员
~~~
namespace A {
	int m_B = 20;
}
~~~


- 匿名命名空间，意味着命名空间中的标识符只能在本文件内访问，相当于给这个标识符加上了static，使得其可以作为内部连接。

- 可以起别名
~~~
namespace S = A;
~~~

### namespace用途以及使用方法

1. using声明:
using声明可使得指定的标识符可用。
2. using编译指令:
using编译指令使整个命名空间标识符可用.


参照上方的命名空间A和B进行使用表示using的使用注意事项。
~~~
void test() {

	int m_A = 100;

	//声明，注意二义性问题
	//using A::m_A;//编译会失败

	//编译指令，打开房间，并不一定用。
	using namespace A;
	//using namespace B;//连续打开两个房间，一会产生二义性。

	cout << m_A << endl; //此时输出结果为100，虽然此时打开了A的房间，但是并未使用。

}
~~~


## 引用（reference）

引用是C++对C的重要扩充。在C/C++中指针的作用基本都是一样的，但是C++增加了另外一种给函数传递地址的途径，这就是**按引用传递**(pass-by-reference)

-	变量名实质上是一段连续内存空间的别名，是一个标号(门牌号)
-	程序中通过变量来申请并命名内存空间
-	通过变量的名字可以使用存储空间

**定义方式：Type& ref = val;**

注意事项：
-	&在此不是求地址运算，而是起标识作用。
-	类型标识符是指目标变量的类型
-	**必须在声明引用变量时进行初始化**。
-	**引用初始化之后不能改变**。
-	不能有NULL引用。必须确保引用是和一块**合法的存储单元**关联。即不要返回局部变量的引用
-	可以建立对数组的引用。

定义数组的两种方式
~~~
typedef int(ARRAY)[10];
void test09() {
	int arr[10];

	int(&reArr)[10] = arr;

	ARRAY& r = arr;

}
~~~

做函数参数，例如交换函数，C中调用时需要传入地址，参数需要指针类型，C++大大简化了这些操作
~~~
void swap(int& a, int& b) {
	int temp = a;
	a = b;
	b = temp;
}

void test10() {
	int a = 10;
	int b = 20;
	swap(a, b);
}
~~~

做函数返回值时
~~~
int& test01() {
	static int a = 10;
	return a;
}

void test(){
	test01() = 100;
}
~~~

### 引用的本质
**引用的本质在c++内部实现是一个指针常量**

**自动转换为val的地址或值。**
~~~
Type& ref = val; // Type* const ref = &val;
~~~

###  指针引用
在C中如果想改变一个指针的指向而不是它所指向的内容，需要用到二级指针，使用指针引用会让函数清晰明了很多。
~~~
Type* pointer = NULL;  
Type*& = pointer;
~~~

两者的具体参考以下的代码
~~~
struct Teacher{
	int mAge;
};

//指针间接修改teacher的年龄
void AllocateAndInitByPointer(Teacher** teacher){
	*teacher = (Teacher*)malloc(sizeof(Teacher));
	(*teacher)->mAge = 200;  
}

//引用修改teacher年龄
void AllocateAndInitByReference(Teacher*& teacher){
	teacher->mAge = 300;
}

void test(){
	//创建Teacher
	Teacher* teacher = NULL;
	//指针间接赋值
	AllocateAndInitByPointer(&teacher);
	cout << "AllocateAndInitByPointer:" << teacher->mAge << endl;
	//引用赋值,将teacher本身传到ChangeAgeByReference函数中
	AllocateAndInitByReference(teacher);
	cout << "AllocateAndInitByReference:" << teacher->mAge << endl;
	free(teacher);
}
~~~

**常量引用**主要用在函数的形参，尤其是类的拷贝/复制构造函数。

将函数的形参定义为常量引用的好处:
- 引用不产生新的变量，减少形参与实参传递时的开销。
- 由于引用可能导致实参随形参改变而改变，将其定义为常量引用可以消除这种副作用。
- 如果希望实参随着形参的改变而改变，那么使用一般的引用，如果不希望实参随着形参改变，那么使用常量引用。

PS:起床后不是立马学习，而是选电脑选了半天。。。