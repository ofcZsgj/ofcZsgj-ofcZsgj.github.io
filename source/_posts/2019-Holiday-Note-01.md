---
title: Holiday Note_01
date: 2019-07-13 17:54:39
categories: C
tags: [Note,C]
---

# Holiday Note_01

今天是学校放假的第一天，趁着这个暑假，把丢了一个学期的博客捡起来。每晚花点时间总结下白天学到的知识点，这样不至于出现学一天忘两天的尴尬情况，也方便以后复习，目前计划是一天一篇📒📘📗📕📔📙，以上。

<!-- more -->

先做个总结，今天学的不算是新知识了，大都是对以前学过知识的拓展补充因此学习起来难度并不是很大，比如数据类型的一些新使用场景和方式，自增语句以及判断语句 (++i，if(NULL == p)) 的优化写法，内存四区模型的拓展，栈区和堆区使用的注意要点（堆区的使用还是挺有讲究的这是一个重难点尤其是二级指针作为参数去操作时，使用不当，不仅内存泄露，还会出现野指针），其中关于栈和函数调用流程特别重要，写程序一定要清楚的知道自己写的哪些数据分别在哪个区内，什么时候会被回收，否则可能带来非常多的bug，最后就是指针的步长（这一块比较难以理解，而且是重点，理解后就可以写出各种花里胡哨的指针表示了哈哈），新学到的不过就是宏函数和调用惯例难度不大也不是重点了。

---

## 数据类型
### typedef的新用法
~~~
    char *p1,p2;
    //此写法将只有p1为指针类型，p2仍为char类型
    
    //解决方法如下
    typedef char * pchar;
    pchar p3,p4,p5;

    //且使用typedef有利于程序的移植性
~~~

### void *
~~~
    /*
        1、是所有类型指针的祖宗，又称万能指针
        2、任何类型的指针都不需要经过强制转换为void*
        3、主要用于数据结构的封装,基本上封装的函数参数均为void*
    */
~~~

### unsigned int
~~~
    /*
        1、sizeof()返回的值是unsigned int
        2、一般情况，unsigned int与int进行运算得到的是前者,如 int i=10,j=20; i-j结果为10
        3、切记数组在作为函数参数时会退化为指向数组首元素的指针，当使用sizeof() 求大小时，求得的值实际为指针占用大小
    */
~~~

### 内存分区的的一些补充
- 数据区包括：堆，栈，全局/静态存储区。
- 全局/静态存储区包括：常量区，全局区、静态区。
- 常量区包括：字符串常量区、常变量区。
- 代码区：存放程序编译后的二进制代码，不可寻址区。

可以说，C/C++内存分区其实只有两个，即代码区和数据区。且全局const变量，字符串常量于常量区，无法通过直接或间接修改，局部const位于栈区，可以通过间接修改

---

## 内存分区知识点

### 程序运行前
首先需要了解，C程序在运行之前，要经过如下所示四个步骤:

|步骤|内容|
|:-:|:-:|
|预处理|宏定义展开、头文件展开、条件编译，这里并不会检查语法|
|编译|检查语法，将预处理后文件编译生成汇编文件|
|汇编|将汇编文件生成目标文件(二进制文件)|
|链接|将目标文件链接为可执行程序|

生成可执行性文件后在Linux下可以通过size命令查看可执行性程序的基本情况

即在程序为被加载到内存中前，程序内部已经分好了三段区域
- 代码区（text）注意代码区通常是只读的，共享的。其可共享的目的是对于频繁被执行的程序，只需要在内存中有一份代码即可。
- 数据区（data）该区包含了在程序中明确被初始化的全局变量、已经初始化的静态变量（包括全局静态变量和局部静态变量）和常量数据（如字符串常量）。
- 未初始化数据区（bss）存入的是全局未初始化变量和未初始化静态变量。未初始化数据区的数据在程序开始执行之前被内核初始化为 0 或者空（NULL）。

### 程序运行后
程序运行之后，代码区，数据区以及未初始化数据区的大小就是固定的，程序运行期间不能改变。然后，运行可执行程序，操作系统把物理硬盘程序load(加载)到内存，除了根据可执行程序的信息分出代码区（text）、数据区（data）和未初始化数据区（bss）之外，还额外增加了栈区、堆区。

#### 栈区（stack）
栈区使用不当可能会造成程序运行结果出错，以下为两个错误示例，需要十分注意，
~~~
//栈区内存自动释放，不需要程序手动管理，

int * myFunc() {
	int a = 10;//临时变量a存于栈上
	return &a;
}


//不要返回局部变量的地址
void test01() {
	int* p = myFunc();
	printf("%d\n", *p);
}

char *getString() {
	char str[] = "Hello World!";//str在getString()结束后被销毁
	return str;
}

void test02() {
	char* s = NULL;
	s = getString();
	printf("s=%s", s);
}

int main() {
	//此使已经不关心值是多少，因为局部变量a的内存空间已经被回收
	test01();

	//Hello World!被存入常量区，但是str存在与栈区，getString（）结束后被回收，str地址存放值不再是HelloWorld
	test02();
}
~~~
以下为栈的生长方向示意图，向低地址生长，压栈（push）使得栈顶元素的地址变小，出栈（pop）使得栈顶元素的地址变大。
![8edce2a9aae27a724903e.jpg](https://miao.su/images/2019/07/13/8edce2a9aae27a724903e.jpg)

~~~
//栈和内存生长方式的理解代码
//1. 栈的生长方向
void test01(){

	int a = 10;
	int b = 20;
	int c = 30;
	int d = 40;

	printf("a = %d\n", &a);
	printf("b = %d\n", &b);
	printf("c = %d\n", &c);
	printf("d = %d\n", &d);

	//a的地址大于b的地址，故而生长方向向下
}

//2. 内存生长方向(小端模式)
void test02(){
	
	//高位字节 -> 地位字节
	int num = 0xaabbccdd;
	unsigned char* p = &num;

	//从首地址开始的第一个字节
	printf("%x\n",*p);
	printf("%x\n", *(p + 1));
	printf("%x\n", *(p + 2));

~~~

### 堆区（heap）
以下为堆区使用的两个错误例子，导致了内存泄露，且堆释放不当还可能造成野指针，因此需要十分注意！
~~~
int* getSpace() {
	int* p = malloc(sizeof(int) * 5);//p是在栈上，但是malloc分配的20个字节在堆上

	if (NULL == p) {//该写法NULL放前边防止出现写成p=NULL且不报错的情况
		return NULL;
	}
	 //只要是连续分配的内存空间都可以用下标来访问
	for (int i = 0; i < 5; ++i) {//++i效率高
		p[i] = 100 + i;
	}
	return p;
}

void test03() {
	char* p = NULL;
	allocateSpace(p); //执行完毕后，p仍然是NULL
	printf("p= %s", p);//打印结果为NULL，造成了内存泄露！！！
}
~~~

~~~
void allocateSpace(char* p) {
	p = malloc(100);//开辟了堆空间，p指向的堆空间存放了HelloWorld
	memset(p, 0, 100);
	strcpy(p, "Hello World");//p存放与栈上，结束后销毁，堆空间的地址从此 未知!
}

void test04() {
	char* p = NULL;
	allocateSpace(p); //执行完毕后，p仍然是NULL
	printf("p= %s", p);//打印结果为NULL，造成了内存泄露！！！
}
~~~

要想正确使用堆区，关键在于找到分配的堆空间的地址，一定不能丢掉开辟堆空间的地址，以下示例使用了二级指针进行操作

~~~
void allocateSpace02(char** p) {
	char* temp = malloc(100);
	memset(temp, 0, 100);
	strcpy(temp, "Hello World");
	//关键步骤！
    *p = temp;
    //让传递过来的p指针指向开辟堆空间的地址，因此结束释放了形参char**p和临时变量char*temp也就无所谓了
}

void test05() {
	char* p = NULL;
	allocateSpace02(&p);
	printf("p= %s\n", p);
    
	free(p);
	p = NULL;//free(p) 仅仅只是将该堆空间释放，p若不置为空则成为野指针
}
~~~
以下为allocateSpace02函数内存分配示意图为辅助理解
![heap21e261.png](https://miao.su/images/2019/07/13/heap21e261.png)

---

## 函数调用模型
函数调用流程示意图如下所示 (十分重要，写代码时对于每一条语句的执行心中都要有此模型)：

[![001dbea1.md.jpg](https://miao.su/images/2019/07/13/001dbea1.md.jpg)](https://miao.su/image/TNrPP)
[![002b3804.md.jpg](https://miao.su/images/2019/07/13/002b3804.md.jpg)](https://miao.su/image/TNEdB)
[![00364141.md.jpg](https://miao.su/images/2019/07/13/00364141.md.jpg)](https://miao.su/image/TNJ5J)
[![004f551d.md.jpg](https://miao.su/images/2019/07/13/004f551d.md.jpg)](https://miao.su/image/TNdUm)
[![00575dc4.md.jpg](https://miao.su/images/2019/07/13/00575dc4.md.jpg)](https://miao.su/image/TNxKs)

### 函数调用流程
栈在程序运行中具有极其重要的地位。最重要的，栈保存一个函数调用所需要维护的信息，这通常被称为堆栈帧(Stack Frame)或者活动记录(Activate Record).一个函数调用过程所需要的信息一般包括以下几个方面：
- 函数的返回地址；
- 函数的参数；
- 临时变量；
- 保存的上下文：包括在函数调用前后需要保持不变的寄存器

### 调用惯例
~~~
/*  
	根据下面的代码衍生出两个问题：
	1、参数进行压栈时，对于参数的选择是从左到右还是从右到左 
	2、函数参数的出栈是在func内进行还是在main中进行
*/

/*
	因此需要一个惯例来进行规定以下几个内容，C语言默认为_cdecl调用惯例   
	1、出栈方: 为调用函数方，即若main函数调用func函数则由main函数将参数进行出栈操作
	2、参数传值次序: 由右到左压栈
	3、名字修饰方式: 下划线加函数名
*/

// _cdecl默认不用加
int _cdecl func(int a, int b) {
	int t_a = a;
	int t_b = b;
	return t_a + t_b;
}

int main() {
	int ret = 0;
	ret = func(10, 20);
	int  a = 0;
	return 0;
}
~~~

---

## 指针的步长（通过实际代码进行理解）
以下通过两个示例从易到难演示指针的步长。
~~~
/*
	 重难点！！！
	 指针的步长：指针变量+1，要向后跳多少个字节
*/

void test06() {
	char buf[1024];
	int a = 100;
	memset(buf, 0, 1024);
	memcpy(buf + 1, &a, sizeof(int));

    //以下为重难点！ 
    //buf为char类型，p+1跳1字节，通过强转使得p可以只跳1字节从而取到存放a的首地址后
	char* p = buf;	
	//由于a是int类型，需要转换int*取出存放a的4个字节，再用*取出这四个字节为a的值
	printf("a= %d\n", *(int*)(p + 1));
}
~~~

~~~
#include<stddef.h>

struct Person {
	int a;
	char b;
	char buf[64];
	int d;
};

void test07() {
	struct Person p = { 10,'b',"Hello World",100 };
	//offsetof()是宏函数，包含再stddef.h头文件中，计算结构体中变量偏移位置
	//printf("b off:%d\n", offsetof(struct Person, b));

	//使用Person类型指针取出d的值,重难点！！！！！！
	printf("d= %d\n", *(int*)((char*)(&p) + offsetof(struct Person, d)));
	
    /*  解析如下：
		1、&p 步长为Person的字节数
		2、(char*)(&p) 步长转换为1
		3、((char*)(&p) + offsetof(struct Person, d))  通过offsetof()求出d的偏移量并指向d的首地址
		4、(int*)((char*)(&p) + offsetof(struct Person, d))   d为int类型，使用int*获取d的四个字节地址
		5、*取出内存存放的d的值
	*/

}
~~~

---

##  宏函数
宏函数不是一个真正的函数，只是预处理时做简单的文本替换
~~~
#define MYADD(x,y) ((x)+(y))

//宏函数在一定的场景下比普通函数的运行效率要高
//以空间换取时间
//对于频繁调用的简单函数，可以使用宏函数代替，因为宏函数没有普通函数调用的开销（函数压栈，跳转，返回等）

int main() {
	int a = 10, b = 20;
	//MYADD(a,b)实际只是被替换成了（（a）+（b））
	printf("a + b = %d\n", MYADD(a, b));
}
~~~

PS:太久没捣腾hexo了，命令都忘干净了，来来回回切换看效果花了好久，开头写的每晚花一点时间，今晚就整了快三个小时我也是醉了。。。Markdown语法也是只记得些简单的，写完一看网页效果看起来很不咋样，还写得啰嗦的要死，BUT，先这样吧，至少开了个头。