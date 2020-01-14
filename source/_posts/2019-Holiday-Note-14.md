---
title: Holiday Note_14
date: 2019-07-26 21:18:57
categories: C++
tags: [Note,C++]
---
# Holiday Note_14

学习了C++中两个重要的知识点：**运算符重载**以及**继承**。其中运算符的重载特性真的是非常的灵活，C++中几乎所有的运算符都可以被赋予新的功能。它的本质还是**函数调用**，使用得当可以让涉及类的代码更加的**易于读写**；简单继承派生还是挺好理解的，就是一个父与子的关系。不过**多继承**，**菱形继承**还有**虚继承**的实现原理这块不太好理解，明天再对这一块深入学习。

<!-- more -->

---

## 运算符重载(operator overloading)

[![operator09faf.md.png](https://miao.su/images/2019/07/26/operator09faf.md.png)](https://miao.su/image/T2Xk6)

先总结一下要注意的点：
-	=, [], () 和 -> 操作符只能通过成员函数进行重载 
-	<< 和 >>只能通过全局函数配合友元函数进行重载 
-	不要重载 && 和 || 操作符，因为无法实现短路规则


运算符重载，就是**对已有的运算符重新进行定义**，赋予其另一种功能，以**适应不同的数据类型**。

由于C++有类的概念，因此在使用普通运算符对类进行操作时，会由于识别不了而无法达到像操作简单数据类型一样操作类了，引入运算符重载将运算符赋予对某一个类进行特定的操作达到代码的易于读写的目的。

---

### 介绍案例
有这样一个类
~~~
class Person{
public:

	Person(int age, string name){
		m_Age = age;
		m_Name = name;
	}	

	int m_Age;
	string m_Name;
}
~~~
实例两个对象
~~~
Person p1(7, "小明");
Person p2(9, "小红");
~~~
假设在没有进行运算符重载时，判断两个类的成员是否相同要对类的单个成员进行比对判断。
~~~
if(p1.m_Age == p2.m_Age && p1.m_Name == p2.m_Name){
	cout << "相同" << endl;
}
else cout << "不同" << end;;
~~~
若是使用操作符重载，将逻辑运算符==进行重载
~~~
if(p1 == p2){
	cout << "相同" << endl;
}
else cout << "不同" << end;;
~~~
可见，通过操作运算符重载，可以使得原本无法对类进行判断的==运算符拥有了我们自己需要的判断功能，这便是运算符重载带来了的便利。

---

### +
对+进行重载演示写法

类中包含成员函数方式实现运算符重载1
~~~
class S {

public:

	S() {}
	S(int a, int b) : m_A(a), m_B(b) {}

	int m_A;
	int m_B;
	
	//重载函数
	S operator+(S& s) {
		S temp;
		temp.m_A = this->m_A + s.m_A;
		temp.m_B = this->m_B + s.m_B;
		return temp;
	}

};
~~~
全局函数方式实现运算符重载2，由于参数不同，可以让+有不同的功能
~~~
//重载函数
S operator+(S& s1, int val) {
	S temp;
	temp.m_A = s1.m_A + val;
	temp.m_B = s1.m_B + val;
	return temp;
}
~~~
实现效果
~~~
void test() {

	S s1(1, 1);
	S s2(2, 2);

	S s3 = s1 + s2;
	S s4 = s1 + 10;

	cout << s3.m_A << endl;//3
	cout << s3.m_B << endl;//3
	cout << s4.m_A << endl;//11
	cout << s4.m_B << endl;//11

}
~~~

---

### <<
对左移运算符重载**必须使用全局函数定义方式定义而不能使用成员函数方式定义**。

cout的类型为ostream
~~~
class Person {

	friend ostream& operator<<(ostream& cout, Person& p);

public:

	Person(int a, int b) {
		this->m_A = a;
		this->m_B = b;
	}

private:
	int m_A;
	int m_B;

};
~~~
返回ostream&目的是可以继续实现<<链式编程
~~~
ostream& operator<<(ostream&cout, Person& p) {
	cout << "m_A = " << p.m_A << " m_B = " << p.m_B;
	return cout;
}

~~~
实现效果
~~~
void test() {

	Person p1(7, 9);

	cout << p1 << endl;//m_A = 7 m_B = 9

}
~~~

---

### >>

与重载<<类似，对>>进行重载也只能使用全局函数方式进行实现。

cin的类型为istream

类的定义
~~~
class Test {
public:
	Test(const char* s) {
		this->str = new char[strlen(s) + 1];
		strcpy(str, s);
	}

	char* str;
};
~~~
判断需要录入数据的变量是否有数据，有要先进行释放置空操作。
~~~
istream& operator>>(istream& cin, Test& s) {
	if (s.str != NULL) {
		delete s.str;
		s.str = NULL;
	}

	char buf[1024];
	cin >> buf;

	s.str = new char[strlen(buf) + 1];
	strcpy(s.str, buf);
	return cin;
}
~~~
实现效果
~~~
Test t1("AAA");
cout << t1.str << endl;
cin >> t1;
cout << t1.str << endl;
~~~

---

### =
必须使用成员函数方法实现**=**的重载
~~~
class Person {

public:
	Person(const char* name) {
		this->pName = new char[strlen(name) + 1];
		strcpy(pName, name);
	}

	Person& operator=(const Person& p) {

		if (this->pName != NULL) {
			delete[] this->pName;
			pName = NULL;
		}

		pName = new char[strlen(p.pName) + 1];
		strcpy(pName, p.pName);

		return *this;

	}

	~Person() {
		if (this->pName != NULL) {
			delete[] this->pName;
			pName = NULL;
		}
	}

	char* pName;

};

~~~
实现效果
~~~
Person p1("AAA");
Person p2("BBB");
Person p3("CCC");

p3 = p2 = p1;
~~~


---

### ==
前面的示例即为**==**的重载，实现过程无非就是把原本要写的判断放到运算符重载函数中。

注意的是：对关系运算符进行重载时，不要对**&&**，**||**进行重载！

因为内置版本是具备**短路特性**的，而我们进行重载时是无法实现的。

---

### ++

前置++效率更高，后置++由于返回的是临时变量，因此无法返回引用所以效率相对前置++效率会低。

~~~
class MyInterger {
public:
	MyInterger() {
		m_A = 0;
	}

	//前置++返回引用，效率高
	MyInterger& operator++() {
		this->m_A++;
		return *this;
	}

	//后置++返回值，因为temp为临时变量，不能返回引用
	MyInterger operator++(int) {
		MyInterger temp = *this;
		this->m_A++;
		return temp;
	}

	int m_A;
};
~~~
实现效果
~~~
++(++myInt);//myInt.m_A = 2
~~~


---

### * 和 ->

这是一个智能指针思想的例子。使用一个智能指针类来维护一个需要开辟空间的类。目的是防止new完后忘记delete。（new后，只有手动delete才会释放对象开辟的空间且进行析构函数调用）

智能指针对象在栈上创建，生命周期结束一定会调用析构函数，因此保证维护的类空间能保证释放。

被智能指针类维护的类
~~~
public:

	Person(int a) {
		this->m_A = a;
	}

	void show() {
		cout << this->m_A << endl;
	}

	~Person() {}

	int m_A;
};
~~~

智能指针类，通过对 * 和 -> 的重载，达到SmartPoint既能保证Person类new空间的释放，还能达到指针的特性。
~~~
class SmartPoint {

public:

	SmartPoint(Person* person) {
		this->person = person;
	}

	Person* operator->() {

		return this->person;

	}

	Person& operator*() {

		return *this->person;

	}

	~SmartPoint() {
		if (this->person != NULL) {
			delete(person);
			person = NULL;
		}
	}

private:
	Person* person;

};
~~~
实现效果
~~~
SmartPoint sp = SmartPoint(new Person(10));

sp->show();//sp->->show();

(*sp).show();
~~~

---

### ()

**仿函数**

对函数调用运算符重载，只是简单的接触了用法，可以让对象像一个函数调用。

简单案例
~~~
class Add {

public:
	Add() {

	}

	Add(int a) {
		this->m_A = a;
	}

	int operator()(int a, int b) {
		return a + b;
	}

private:
	int m_A;
};

~~~
实现效果
~~~
Add add1;
cout << add1(1, 1) << endl;//2
~~~


PS:轮子哥写的文章是有味道的😂