---
title: Holiday Note_04
date: 2019-07-16 21:14:31
categories: C
tags: [Note,C]
---

# Holiday Note_04

放假第四天。今天学了**内存对齐**的原理与一系列原则(感觉还是挺简单的，主要突出了时间换空间的思想)，还有**文件打开，关闭以及缓冲区的一些补充知识**。另外写了两个练习，一个是**结构体嵌套二维指针**(依旧是练习分配释放内存，只不过这个练习以Teacher-Student模型分配了4层内存，内存的分配和释放操作还是挺细节的)。另一个练习是 **配置文件读写**案例，巨恶心，写头文件，然后将功能一一封装实现，最后调用测试功能，从下午写到晚上，晕头转向的，全是二级指针三级指针以及结构体指针，刚刚又好好想了一遍，为什么debug了这么久，总结了下是自己没理清楚参数传递的思路和调用过程就写，这样下来效率自然就低了，下次写这种复杂的案例时还是应该先画好图理清楚了再开始。


<!-- more -->

---

## 内存对齐


### 什么是内存对齐

在用sizeof运算符求算某结构体所占空间时，并不是简单地将结构体中所有元素各自占的空间相加，
这里涉及到内存字节对齐的问题。

~~从理论上讲，对于任何变量的访问都可以从任何地址开始访问~~

但是事实上不是如此，**实际上访问特定类型的变量只能在特定的地址访问**，
这就需要各个变量在空间上按一定的规则排列， 而不是简单地顺序排列，这就是内存对齐。

我们知道内存的最小单元是一个字节，当cpu从内存中读取数据的时候，是一个一个字节读取

但是实际上cpu将内存当成多个块，每次从内存中读取一个块，这个块的大小可能是2、4、8、16等，

***

### 内存对齐的优缺点

> 空间换时间

内存对齐是操作系统为了**提高访问内存的策略**。**操作系统在访问内存的时候，每次读取一定长度(这个长度是操作系统默认的对齐数，或者默认对齐数的整数倍)**。如果没有对齐，为了访问一个变量可能产生*二次访问*。而且要注意的是某些平台只能在特定的地址处访问特定类型的数据，否则抛出硬件异常给操作系统。

---

### 内存对齐的三大原则

- 对于*标准数据类型*，它的地址只要是它的**长度的整数倍**。


- 对于非标准数据类型，比如结构体，要遵循以下对齐原则：

1. 数组成员对齐规则。第一个数组成员应该放在offset为0的地方，以后每个数组成员应该放在**offset为min（当前成员的大小，#pargama pack(n)）整数倍**的地方开始（比如int在32位机器为４字节，#pargama pack(2)，那么从2的倍数地方开始存储）。

2. **结构体总的大小**，也就是sizeof的结果，必须是min（结构体内部最大成员，#pargama pack(n)）的整数倍，**不足要补齐**。

3. 结构体做为成员的对齐规则。如果一个结构体B里嵌套另一个结构体A,还是以最大成员类型的大小对齐，但是**结构体A的起点为A内部最大成员的整数倍的地方**。（struct B里存有struct A，A里有char，int，double等成员，那A应该从8的整数倍开始存储。），结构体A中的成员的对齐规则仍满足原则1、原则2。


辅助理解代码如下：

下面这条代码是显示当前packing alignment的字节数，以warning message的形式被显示。我在VS上显示的结果是8.
~~~
#pragma pack(show)
~~~
[![463d913c5f9d055218431668f3e527249333.md.png](https://miao.su/images/2019/07/16/463d913c5f9d055218431668f3e527249333.md.png)](https://miao.su/image/TaHgL)

~~~
typedef struct _STUDENT1 {
	int a;//0-3
	char b;//4
	double c;//8-15  每个成员必须放在min（当前成员的大小，#pargama pack(n)）整数倍的地方开始
	float d;//16-19   结构体总的大小必须为min（结构体内部最大成员，#pargama pack(n)）为8的整数倍，因此0-23  大小一共为24
}Student1;


typedef struct _STUDENT2 {
	char a;//0
	Student1 b;//8-31 结构体A的起点为A内部最大成员的整数倍的地方
	double c;//32-39   0-39  40字节
}Student2;
~~~

~~~
void test() {

	printf("%d\n", sizeof(Student1));//输出结果为24
	printf("%d\n", sizeof(Student2));//输出结果为40

}
~~~

---

## 结构体嵌套二维指针练习

![1205c38d33e627257b4b4.png](https://miao.su/images/2019/07/16/1205c38d33e627257b4b4.png)

结构体如下所示，模型为一个Teacher类型的二级指针，为它分配存放三个Teacher类型的一个指针类型用于存放数据，这三个Teacher类型的一级指针再为name和students分配空间，其中students为二级指针即表示有多个学生，因此使用以及分配的二维指针students继续分配4个一维指针存放学生姓名。该案例关键在于**训练分配内存和释放内存的次序以及判定条件的书写**。
~~~
struct Teacher {
	char* name;
	char** students;
};
~~~

~~~
void printTeacher(struct Teacher** ts) {
	for (int i = 0; i < 3; ++i) {

		printf("%s\n", ts[i]->name);

		for (int j = 0; j < 5; ++j) {

			printf("%s\n", ts[i]->students[j]);

		}
	}
}
~~~

分配内存
~~~
int allocateSpace(struct Teacher*** ts) {
	
	if (NULL == ts)
		return -1;
	
	struct Teacher** temp = malloc(sizeof(struct Teacher*) * 3);

	for (int i = 0; i < 3; ++i) {

		temp[i] = malloc(sizeof(struct Teacher));

		temp[i]->name = malloc(sizeof(char) * 64);

		sprintf(temp[i]->name, "Teacher_%d", i + 1);

		temp[i]->students = malloc(sizeof(char*) * 5);

		for (int j = 0; j < 5; ++j) {
			temp[i]->students[j] = malloc(sizeof(char) * 64);
			sprintf(temp[i]->students[j], "\tStudent_%d", j + 1);
		}

	}

	//指针的间接赋值
	*ts = temp;

	return 0;
}
~~~
**释放内存(关键)**
~~~
void freeSpace(struct Teacher** ts) {

	if (NULL == ts)
		return;

	for (int i = 0; i < 3; ++i) {

		if (NULL == ts[i])
			continue;

		if (ts[i]->name != NULL) {
			free(ts[i]->name);
			ts[i]->name = NULL;
		}

		if (ts[i]->students != NULL) {

			for (int j = 0; j < 5; ++j) {

				if (NULL == ts[i]->students[j]) {
					continue;
				}

				free(ts[i]->students[j]);
				ts[i]->students[j] = NULL;

			}

			free(ts[i]->students);
			ts[i]->students = NULL;

		}

		free(ts[i]);
		ts[i] = NULL;

	}

	free(ts);
	ts = NULL;

}
~~~
调用函数
~~~
void test() {

	struct Teacher** teacher;

	teacher = NULL;

	//定义allocateSpace函数返回值类型为int的目的为接收不同情况的错误码
	printf("%d\n", allocateSpace(&teacher));

	printTeacher(teacher);

	freeSpace(teacher);

}


int main() {
	test();
}
~~~

---

## 文件相关

### 文件缓冲区

如我们从磁盘里取信息，我们先把读出的数据放在缓冲区，计算机再直接从缓冲区中取数据，等缓冲区的数据取完后再去磁盘中读取，这样就可以**减少磁盘的读写次数**，再加上计算机对缓冲区的操作大大快于对磁盘的操作，故应用缓冲区可大大**提高计算机的运行速度**。

![566664cfae.png](https://miao.su/images/2019/07/16/566664cfae.png)


---

### 理解fopen与fclose与文件缓冲区的关系
文件的打开操作表示将给用户指定的文件在内存**分配一个FILE结构区**，并将该结构的指针返回给用户程序，以后用户程序就可用此FILE指针来实现对指定文件的存取操作了。

*只有对打开的文件进行关闭操作时，停留在文件缓冲区的内容才能写到该文件中去*，从而使文件完整。再者一旦关闭了文件，该文件对应的FILE结构将被释放，从而使关闭的文件得到保护，因为这时对该文件的存取操作将不会进行。**文件的关闭也意味着释放了该文件的缓冲区**。

---

### 有关文件指针和EOF的注意事项
关键在于文件指针何时指向EOF：只有当进行fgetc操作后文件指针后才会指向EOF，此使feof的返回值才为真，没进行fgetc操作前，文件指针仍未指向EOF。若不注意可能在读取时将EOF读取。
~~~
void test01() {

	FILE* fp = fopen("./test.txt", "r");

	char ch;
	while (!feof(fp)) {//注意：该写法将会读取出EOF，因为只有当进行了fgetc操作，文件指针才指向EOF，FILE 中的flag值才会改变，此刻的feof(fp)返回值才为真。

		ch = fgetc(fp);

		//加入判断可避免读取出EOF，进行完fgetc操作后文件指针已经指向了EOF
		if (feof(fp))
			break;

		printf("%c", ch);

	}

	fclose(fp);
	fp = NULL;

}
~~~
下面的写法可以直接规避打印EOF的情况
~~~
void test02() {

	FILE* fp = fopen("./test.txt", "r");

	char ch;
	while ((ch = fgetc(fp)) != EOF) {
		printf("%c", ch);
	}

	fclose(fp);
	fp = NULL;
}
~~~

---


### 配置文件读写代码

思路如下: 有如下的配置文件，现需要按找指定格式读取其中有用的key值和value值，如ip:192.168.137.134，ip地址对应key值，192.168.137.134对应value值。将文件中所有有效的key，value值读取后为其分配对应的内存空间，并存放与结构体中，且之后可以通过给定一个key值返回对应的value值。

![359febb5d031e91e0bb5f0abb4c31abca5ff.png](https://miao.su/images/2019/07/16/359febb5d031e91e0bb5f0abb4c31abca5ff.png)

以下是头文件，定义好函数原型以及存储key，value的结构体，实现过程有点复杂，但是更重要的是**理解各个函数调用的过程以及参数的传递作用**！！！
~~~
//防止头文件重复包含
#pragma once

#include<stdio.h>
#include<stdlib.h>
#include<string.h>

//为了C++能够调用C写的函数
#ifdef __cplusplus
	extern "C" {
#endif

	struct ConfigInfo {
		char key[64];
		char value[128];
	};

	//获取文件有效行
	int getLines_ConfigFile(FILE* file);

	//加载配置文件
	void loadFile_ConfigFile(const char* filePath, char*** fileData, int* line);

	//解析配置文件
	void parseFile_ConfigFile(const char** fileData, int line, struct ConfigInfo** info);

	//获得指定配置信息
	void getInfo_ConfigFile(const char* key, struct ConfigInfo* info, int line);

	//释放配置文件信息
	void destroyFile_ConfigFile(struct ConfigInfo* info);

	//判断当前行是否有效
	int isValid_ConfigFile(const char* buf);


#ifdef __cplusplus
	}
#endif 
~~~

以下为功能实现文件

获取文件有效行
~~~
#include "ConfigFile.h"

int getLines_ConfigFile(FILE* file) {

	char buf[1024] = { 0 };
	int lines = 0;
	
	while (fgets(buf, 1024, file) != NULL) {
		
		if (!isValid_ConfigFile(buf)) {
			continue;
		}
		
		memset(buf, 0, 1024);

		++lines;
	}

	fseek(file, 0, SEEK_SET);

	return lines;

}
~~~
加载配置文件
~~~
void loadFile_ConfigFile(const char* filePath, char*** fileData, int* line) {

	FILE* file = fopen(filePath, "r");

	if (NULL == file)
		return;

	int lines = getLines_ConfigFile(file);
	char** temp = malloc(sizeof(char*) * lines);

	char buf[1024] = { 0 };
	int index = 0;

	while (fgets(buf, 1024, file) != NULL) {

		if (!isValid_ConfigFile(buf))//返回false时，则该行无效，读取下一行
			continue;

		temp[index] = malloc(sizeof(char) * strlen(buf) + 1);

		strcpy(temp[index], buf);

		++index;

		memset(buf, 0, 1024);
	}

	*fileData = temp;
	*line = lines;

	fclose(file);

}
~~~
解析配置文件
~~~
void parseFile_ConfigFile(const char** fileData, int line, struct ConfigInfo** info) {

	struct ConfigInfo* myInfo = malloc(sizeof(struct ConfigInfo) * line);
	memset(myInfo, 0, sizeof(struct ConfigInfo) * line);

	for (int i = 0; i < line; ++i) {

		char* pos = strchr(fileData[i], ':');

		strncpy(myInfo[i].key, fileData[i], pos - fileData[i]);

		int flag = 0;
		if (fileData[i][strlen(fileData[i]) - 1] == '\n') {//等价于*(fileData[i] + strlen(fileData[i]) - 1);
			//此使代表该行结尾为换行
			flag = 1;
			printf("该行有换行\n");
		}

		strncpy(myInfo[i].value, pos + 1, strlen(pos + 1) - flag);//将有换行的'\n'不进行拷贝

		printf("key = %s  value = %s\n", myInfo[i].key, myInfo[i].value);

	}

//释放文件信息
	for (int i = 0; i < line; ++i) {
		if (fileData[i] != NULL) {
			
			free(fileData[i]);
			fileData[i] = NULL;
		
		}
	}

	*info = myInfo;

}
~~~
获得指定配置信息
~~~
void getInfo_ConfigFile(const char* key, struct ConfigInfo* info, int line) {

	for (int i = 0; i < line; ++i)
	{
		if (strcmp(key, info[i].key) == 0)
		{
			return info[i].value;
		}
	}
~~~
释放配置文件信息
~~~
void destroyFile_ConfigFile(struct ConfigInfo* info) {

	if(NULL == info)
	{
		return;
	}

	free(info);
	info = NULL;

}
~~~
判断当前行是否有效
~~~
int isValid_ConfigFile(const char* buf) {

	if (buf[0] == '#' || buf[0] == '\n' || (strchr(buf, ':') == NULL))
		return 0;

	return 1;

}
~~~
以下为测试功能函数以及主函数文件
~~~
void test() {
	
	char** fileData = NULL;
	int lines = 0;
	struct ConfigInfo* myInfo = NULL;

	loadFile_ConfigFile("./config.ini", &fileData, &lines);

	printf("%d\n", lines);

	parseFile_ConfigFile(fileData, lines, &myInfo);

	destroyFile_ConfigFile(myInfo);

}

int main() {
	test();
}
~~~

PS：多次想放弃，磕磕绊绊看着源码一行一行改错。。。对这种逻辑复杂的配置文件读写案例，code之前切记画图，切记画图，切记画图。不画图，debug时泪流满面。