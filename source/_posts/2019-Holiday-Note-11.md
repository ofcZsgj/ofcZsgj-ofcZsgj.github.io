---
title: Holiday Note_11
date: 2019-07-23 12:16:02
categories: C++
tags: [Note,C++]
---

# Holiday Note_11

学了C++的**内联函数**，就是对C中宏的替代升级。还有**内联函数**，不怎么重要的函数的默认参数和占位参数，今天比较重要的是C++中的**重载**和**类和对象**的概念和使用，暂时觉得和跟上学期学的Java很像，写一堆其实都是get&set方法。。。

<!-- more -->

---

## 内联函数(inline function)

~~~
inline int func(int a){return ++;}
~~~

C++引入内联函数是用于替换C中的宏。

**内联函数为了继承宏函数的效率，没有函数调用时开销，然后又可以像普通函数那样，可以进行参数，返回值类型的安全检查，又可以作为成员函数。内联函数本身也是一个真正的函数。**

- 宏函数会有隐藏一些难以发现的错误。例如括号问题；((a++) < (b)) ? (a++) : (b)。
- 预处理器不允许访问类的成员，也就是说预处理器宏不能用作类类的成员函数

### 需要注意的是：

1. **函数体和声明必须结合**在一起，否则编译器将它作为普通函数来对待。
2. 内联函数是以**空间换时间**
3. 任何**在类内部定义**的函数**自动**成为内联函数

### 内联函数的一些限制
-	不能存在**任何形式的循环语句**
-	不能存在**过多的条件判断**语句
-	函数体不能过于庞大
-	不能**对函数进行取址**操作（类似宏，编译后找不到这个函数的入口）

> 总的来说内联仅仅只是**给编译器一个建议**，编译器不一定会接受这种建议，如果你没有将函数声明为内联函数，那么编译器也可能将此函数做内联编译。一个好的编译器将会内联小的、简单的函数。

---

## 函数的默认参数和占位参数(很少用到的东西)

### 默认参数
C++在声明函数原型的时可为一个或者多个参数指定默认(缺省)的参数值，当函数调用的时候如果没有指定这个值，编译器会自动用默认值代替。
~~~

void TestFunc01(int a = 10, int b = 20){
	cout << "a + b  = " << a + b << endl;
}

//注意点:
//1. 形参b设置默认参数值，那么后面位置的形参c也需要设置默认参数
void TestFunc02(int a,int b = 10,int c = 10){}

//2. 如果函数声明和函数定义分开，函数声明设置了默认参数，函数定义不能再设置默认参数
void TestFunc03(int a = 0,int b = 0);
void TestFunc03(int a, int b){}

int main(){
	//1.如果没有传参数，那么使用默认参数
	TestFunc01();
	//2. 如果传一个参数，那么第二个参数使用默认参数
	TestFunc01(100);
	//3. 如果传入两个参数，那么两个参数都使用我们传入的参数
	TestFunc01(100, 200);

	return EXIT_SUCCESS;
}

~~~

### 函数的占位参数
C++在声明函数时，可以设置占位参数。占位参数只有参数类型声明，而没有参数名声明。一般情况下，在函数体内部无法使用占位参数。
~~~

void TestFunc01(int a,int b,int){
	//函数内部无法使用占位参数
	cout << "a + b = " << a + b << endl;
}

//占位参数也可以设置默认值
void TestFunc02(int a, int b, int = 20){
	//函数内部依旧无法使用占位参数
	cout << "a + b = " << a + b << endl;
}

int main(){

	//错误调用，占位参数也是参数，必须传参数
	//TestFunc01(10,20); 
	//正确调用
	TestFunc01(10,20,30);
	//正确调用
	TestFunc02(10,20);
	//正确调用
	TestFunc02(10, 20, 30);

	return EXIT_SUCCESS;
}

~~~

## 函数重载(overload)

跟之前学习Java时差不多，就是通过函数参数设置的不同从而实现多个同名函数。

### 实现函数重载的条件
-	同一个作用域
-	参数个数不同
-	参数类型不同
-	参数顺序不同

要注意，**返回值不作为函数重载的条件**！
~~~
//1. 函数重载条件
namespace A{
	void MyFunc(){ cout << "无参数!" << endl; }
	void MyFunc(int a){ cout << "a: " << a << endl; }
	void MyFunc(string b){ cout << "b: " << b << endl; }
	void MyFunc(int a, string b){ cout << "a: " << a << " b:" << b << endl;}
    void MyFunc(string b, int a){cout << "a: " << a << " b:" << b << endl;}
}
//2.返回值不作为函数重载依据
namespace B{
	void MyFunc(string b, int a){}
	//int MyFunc(string b, int a){} //无法重载仅按返回值区分的函数
}
~~~

编译器为了实现函数重载，也是默认为我们做了一些幕后的工作，**编译器用不同的参数类型来修饰不同的函数名**，比如void func(); 编译器可能会将函数名修饰成_func，当编译器碰到void func(int x),编译器可能将函数名修饰为_func_int,当编译器碰到void func(int x,char c),编译器可能会将函数名修饰为_func_int_char ，编译器如何修饰重载的函数名称并没有一个统一的标准，不同的编译器可能会产生不同的内部名。

因此，对于引用C的文件，C++必须使用extern命令，否则，C的文件无法引用。(C++编译器支持重载的特性对函数名进行修饰，导致找不到C定义的文件)

在C文件中使用这种格式可以让其中所有函数在C++中无需任何改动使用。
~~~
#ifdef __cplusplus
extern "C"{
#endif

	void func1();

#ifdef __cplusplus
}
#endif
~~~

或者在C++中对每一个C文件中的函数前加上extern "C"。
~~~
extern "C" void func1();
~~~

---

## 类和对象

在C中，行为和属性是分开的。

在C++中，把变量（属性）和函数（操作）合成一个整体，封装在一个类中；对变量和函数进行访问控制（**public**,**protected**,**private**）

- 类中默认private，protected只能内部使用和子类使用，private只能内部使用。

- C++中，class默认访问权限为private,struct默认访问权限为public。

和Java那一套很类似，**将成员变量均设置为private**，通过get，set等成员函数进行查看和修改。看似写了很多行代码，其实全都是很简单的。。。

以下为一个点类的头文件部分定义，头文件记得加#pragma once。
~~~
class Point {
private:
	int m_X;
	int m_Y;
};
~~~
这是实现文件，注意的是，实现时需要加**Point::**进行函数定义。
~~~
int Point::getX() {
	return m_X;
}
~~~

类可以嵌套，如下面这个圆类，圆心定义即为点类。
~~~
class Circle {
private:
	Point m_Center;
	int m_R;
};
~~~

PS:今天点到了一家不错的外卖，难得啊。
