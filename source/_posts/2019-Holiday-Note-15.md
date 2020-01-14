---
title: Holiday Note_15
date: 2019-07-27 20:58:58
categories: C++
tags: [Note,C++]
---

# Holiday Note_15

**继承**，**多态**，**虚基类**，**虚函数**，**纯虚析构函数**，**虚函数指针**，**虚析构函数**，**纯虚析构函数**，还有最搞笑的羊驼(草泥马)——**菱形继承(钻石继承)**🤣。不得不说C++的这些名词取得真有意思。

要自个直接去取虚函数指针找虚函数表里边的函数地址来调用的话，不用Developer Commond prompt看下类的结构真的不好下手。。。
~~~
((void(*)())(*((int*)*(int*)(&dog) + 1)))();
~~~

<!-- more -->

## 继承

使用VS的Developer Commond prompt进入当前工程的目录下，输入命令
~~~	
cl /d1 reportSingleClassLayout类名 文件名
~~~
可以查看类的模型，继承关系，sizeof的大小，虚函数表等。

-	子类会继承父类中所有的内容 ，包括了 私有属性
-	只是我们访问不到，编译器给隐藏了


### 派生类访问控制
[![36deebef62d3c08613e64.md.png](https://miao.su/images/2019/07/27/36deebef62d3c08613e64.md.png)](https://miao.su/image/T7vXU)

### 继承中的构造和析构顺序
-	子类对象在创建时会首先调用父类的构造函数
-	父类构造函数执行完毕后，才会调用子类的构造函数
-	当父类构造函数有参数时，需要在子类初始化列表(参数列表)中显示调用父类构造函数
-	析构函数调用顺序和构造函数相反
- 如果**父类中没有合适默认构造**，那么子类可以利用**初始化列表**的方式显示的调用父类的其他构造

### 继承中同名成员和函数的处理方法
-	当子类成员和父类成员同名时，子类依然从父类继承同名成员
-	如果子类有成员和父类同名，子类访问其成员默认访问子类的成员(本作用域，就近原则)
-	在子类通过作用域::进行同名成员区分(在派生类中使用基类的同名成员，显示使用类名限定符)

- 静态成员和函数的处理方法类似	

### 多继承&&菱形继承
写法
~~~
class A : public B1, public B2,
~~~

避免二义性问题，使用作用域::进行成员和函数的使用。


### 菱形继承
[![cnm88b84.jpg](https://miao.su/images/2019/07/27/cnm88b84.jpg)](https://miao.su/image/T7zO8)

譬如羊驼这种继承羊类和驼类，而羊类和驼类又继承于动物类。以上所有类均有m_Age，则羊驼内部实质有两份m_Age的数据。要**避免数据重复**需要使用到**虚基类**。

在定义羊类和驼类使用**虚基类**的方法。

~~~
class BigBase{
public:
	BigBase(){ mParam = 0; }
	void func(){ cout << "BigBase::func" << endl; }
public:
	int mParam;
};

class Base1 : virtual public BigBase{};
class Base2 : virtual public BigBase{};
class Derived : public Base1, public Base2{}

~~~


[![vbptrbc5e0.png](https://miao.su/images/2019/07/27/vbptrbc5e0.png)](https://miao.su/image/T7oKM)

-	Base1和Base2通过虚继承的方式派生自BigBase,这两个对象的布局图中可以看出编译器为我们的对象中增加了一个vbptr (virtual base pointer),vbptr指向了一张表，这张表保存了当前的虚指针相对于虚基类的首地址的偏移量。
-	Derived派生于Base1和Base2,继承了两个基类的vbptr指针，并调整了vbptr与虚基类的首地址的偏移量找到共有的数据。


## 多态
**父类的引用或指针指向子类对象**

###	静态联编和动态联编
多态分类
-	静态多态  函数重载
-	动态多态 虚函数 继承关系
-	
静态联编
-	地址早绑定 编译阶段绑定好地址

动态联编 
-	地址晚绑定 ，运行时候绑定好地址


### 多态原理解析
-	当父类中有了虚函数后，内部结构就发生了改变
-	内部多了一个 vfprt(virtual  function pointer)**虚函数表指针**指向 vftable  虚函数表
	
-	父类中结构  vfptr     &Animal::speak
-	子类中 进行继承 会继承 vfptr  vftable
-	构造函数中 会将虚函数表指针 指向自己的虚函数表
-	如果**发生了重写**，会**替换掉虚函数表**中的原有的speak，改为 &Cat::speak

### 抽象类和纯虚函数
-	纯虚函数写法  virtual void func() = 0;
-	抽象类型
-	**抽象类** **不可以实例化对象**
-	如果类 继承了抽象类， **必须重写抽象类中的纯虚函数**

###	虚析构和纯虚析构
-	虚析构: virtual ~类名() {}
-	解决问题： 通过父类指针指向子类对象释放时候不干净导致的问题(即子类不会执行析构函数)
-	纯虚析构函数写法  virtual ~类名() = 0 	
-	类内声明  类外实现
-	如果出现了纯虚析构函数，这个类也算抽象类，**不可以实例化对象**

###	向上类型转换和向下类型转换
-	基类转派生类即**向下类型转换**：不安全的	
-	派生类转 基类即向上类型转换：安全	
-	如果发生多态总是安全的

以下为一个虚函数表指针的使用例子

定义父类Animal和子类Dog，使用虚函数。
~~~
class Animal {
public:
	//动态联编，讲speak方法写为虚函数
	virtual void speak() {
		cout << "Animal speak!!!" << endl;
	}
	virtual void eat() {
		cout << "Animal eat!!!" << endl;
	}
};

class Dog :public Animal {
public:
	void speak() {
		cout << "Dog speak!!!" << endl;
	}
	void eat() {
		cout << "Dog eat!!!" << endl;
	}
};
~~~
创建一个Dog类的对象
~~~
Dog dog;
//doSpeak(dog);
~~~
主要操作
~~~
	// 这是函数指针(void(*)())	
	//通过dog对象地址找到虚函数表内的函数地址执行等同doSpeak(dog)的操作
	((void(*)()) * (int*) * (int*)(&dog))();

	// 进阶，虚函数表中有两个函数地址，现在通过编译器方式找到第二个函数的地址并且执行
	((void(*)())(*((int*) * (int*)(&dog) + 1)))();

}
~~~
若父类还有其他变量，则获得虚函数指针时要多进行一次偏移操作。
[![0a74dbfa5e2bed1596b9d9d96b14703548ce.png](https://miao.su/images/2019/07/27/0a74dbfa5e2bed1596b9d9d96b14703548ce.png)](https://miao.su/image/T7RU5)

PS:轮子哥nb