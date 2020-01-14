---
title: Holiday Note_05
date: 2019-07-17 17:26:45
categories: C
tags: [Note,C]
---

# Holiday Note_05

放假第五天，接触了**函数指针**和**回调函数**的概念以及用法。不是那么好理解，但是却是一个异常重要的知识点，视频中表示无论花多少时间精力去理解都是值得的。我也是把代码从不使用回调函数到使用并且以不同方式使用后，才逐渐理解了一些，其思想有点类似与多态，有利于代码的复用以及维护。使用回调函数编写好接口后，其他用户调取时只要注意接口的函数指针类型以及返回值类型，便可以根据自己的业务需求自定义函数内部逻辑了。

然后又重新复习了下**链表**，因为上个寒假我的C还只有三脚猫功夫的时候硬嗑过，再加上这个学期上一些杂课的时候好好读了《数据结构与算法——C语言实现》，这次再实现链表的各种功能时，以前积攒了一堆搞不懂的问题一瞬间迎刃而解的爽快感着实难以形容，真的是大为舒畅。

还有今天早上烧水泡茶的操作把自己迷到了，记在末尾，不过相信这种蠢事一辈子可能都忘不了的了。。。😂


<!-- more -->

---

## 函数指针

**一个函数在编译时被分配一个入口地址，这个地址就称为函数的指针，函数名代表函数的入口地址。**

要记住，函数名只不过是一个函数的**入口地址**，和数组类型指针有些类似，都有**三种方式**进行定义函数指针。



- 1、先定义函数类型
~~~
typedef int(FUNC_TYPE)(int, char);
FUNC_TYPE* pFunc = func;
	
pFunc(10, 'a');
~~~

- 2、定义函数指针类型
~~~
typedef int(*FUNCPOINT_TYPE)(int, char);
FUNCPOINT_TYPE funcPoint = func;

funcPoint(20, 'b');
~~~

- 3、直接定义函数指针变量
~~~
int(*FuncPoint)(int, char) = func;

funcPoint(30, 'c');
~~~

上述函数指针指向的函数原型
~~~
int func(int a, char b) {
	printf("Hello World!\n");
}
~~~

---

## 回调函数

**务必理解该思想，此案例代码一定要读懂。**

以下案例具体可描述为：用户希望将指定的数据进行打印。

因此有这样一个问题：每次给定的数据类型是不固定的，因此在进行数据打印时，无法将函数功能固定死，当然也绝不能将所有类型的数据打印方法分别都实现。因此引入函数回调这样一个思想，假设用户将会**给定怎样的数据我们还不清楚**，我们做的功能仅仅是做好接口，通过用户传递的几个关键参数，我们可以确定每一个数据的地址确定并传出，如果我们提前写好打印方法，那么很明显，若换了另外一种数据，这个函数便无法使用了。因此我们不负责打印。用户根据我们的接口的规则设计相应的函数实现打印功能。接口有一个重要的参数便是函数指针了，用户需要打印何种类型的数据，便编写该类数据的打印函数并将其作为参数传递至接口。这样，在调用我们的接口时，函数指针将会指向用户编写好对应的函数进行功能实现，完成了代码的简化以及复用，大大减少了维护的成本。 



这是编写的**接口函数**，函数指针类型的形参一般使用万能指针类型。
~~~
void printAllArray(void* data, int elementSize, int len, void(*print)(void *)) {

    //传递过来的第一个参数是一个数据数组，第二个参数是每个数据的大小，第三个是数据的个数，第四个是函数指针
	char* start = (char*)data;//将数据转换为字符指针类型来将每一个数据地址求出

	for (int i = 0; i < len; ++i) {
		char* eleAddr = start + i * elementSize;

		//下列语句不能写死，传递过来的数据类型是不确定的，为了代码的复用，应当根据用户需求，使用回调函数
		//printf("%d\n", *((int*)eleAddr));

		//print为函数指针变量，此写法今后不用修改，因为不管用户需要打印何种类型，print都指向了用户自定义的打印函数，printAllArray函数目的是求出需要打印的每个元素的地址
		print(eleAddr);
		
	}

}
~~~

用户自定函数1，用于打印int类型数据
~~~
void myPrintInt(void * eleAdd) {

	//因为这是用户自己写的函数，因此清楚需要转换成指定类型将数据打印出。
	int* pAddress = (int*)eleAdd;
	printf("%d\n", *pAddress);

}
~~~

用户自定函数2，打印struct类型数据
~~~
struct Person {
	char name[64];
	int age;
};

void myPrintPerson(void* dataAddress) {

	struct Person* pAddress = (struct Person*)dataAddress;
	printf("name = %s age = %d\n", pAddress->name, pAddress->age);

}
~~~

可见，接口编写完后，无论打印什么类型的数据，此使打印何种类型数据仅仅是取决用户如何传递的参数信息以及用户对应不同数据针对接口编写的打印函数了
~~~
void test() {
	
	int arr[] = { 1, 2, 3, 4, 5 };

	struct Person persons[5] = {
		{"aaa",10},
		{"bbb",20},
		{"ccc",30},
		{"ddd",40},
		{"eee",50},
	};

	//调用接口打印整型数组类型
	printAllArray(arr, sizeof(arr[0]), 5, myPrintInt);

	printf("--------------------\n");

	//调用接口打印结构体类型
	printAllArray(persons, sizeof(persons[0]), 5, myPrintPerson);

}
~~~

---

## 链表

 > 在《数据结构与算法——C语言描述中》，链表，队列，栈被称为是最简单和最基本的三种数据结构。

### 什么是链表：

-	链表通过指针将一些列数据结点，连接成一个数据链。相对于数组，链表具有更好的**动态性**（*非顺序存储*）。
-	**数据域**用来存储数据，**指针域**用于建立与下一个结点的联系。
-	建立链表时无需预先知道数据总量的，可以随机的分配空间，可以高效的在链表中的任意位置实时插入或删除数据。
-	链表的**开销**，主要是访问顺序性和组织链(指针域)的空间损失。

---

### 数组与链表的区别：

数组：一次性分配一块连续的存储区域。

优点：

- 随机访问元素效率高

缺点：
- 需要分配一块连续的存储区域（很大区域，有可能分配失败）     
- **删除和插入某个元素效率低**

---

链表：无需一次性分配一块连续的存储区域，只需分配n块节点存储区域，通过指针建立关系。

优点：
- 不需要一块连续的存储区域
-  **删除和插入某个元素效率高**

缺点：
- **随机访问**元素效率低


---

### 链表的实现

#### 静态链表
没什么好说的，就是直接定义好，基本不用。
~~~
struct Node {
	int data;
	//指向下一个结点的指针
	struct Node* next;
};


void test() {

	struct Node node1 = {10, NULL };
	struct Node node2 = {20, NULL };
	struct Node node3 = {30, NULL };
	struct Node node4 = {40, NULL };
	struct Node node5 = {50, NULL };
	struct Node node6 = {60, NULL };

	node1.next = &node2;
	node2.next = &node3;
	node3.next = &node4;
	node4.next = &node5;
	node5.next = &node6;

	//辅助指针判断下一个结点是否为空
	struct Node* pCurrent = &node1;
	while (pCurrent != NULL) {

		printf("%d\n", pCurrent->data);

		pCurrent = pCurrent->next;
	}

}
~~~

#### 动态链表的实现

定义头文件，使用最基本的链表结构实现六种基本功能：**初始化，插入，删除，便利，清空，销毁**。
~~~
#pragma once

#include<stdlib.h>
#include<stdio.h>
#include<string.h>
#include<stdbool.h>

#ifdef __cplusplus
extern "C" {
#endif // __cplusplus


	//定义结构体类型
	struct LinkListNode {
		int data;
		struct LinkListNode* next;
	};


	//初始化链表
	struct LinkListNode* Init_LinkList();

	//在值为oldvalue的位置插入一个newvalue
	void InsertByValue_LinkList(struct LinkListNode* header, int oldValue, int newValue);

	//删除值为value的结点
	void RemoveByValue_LinkList(struct LinkListNode* header, int delValue);

	//遍历链表
	void Foreach_LinkList(struct LinkListNode* header);

	//销毁链表
	void Destroy_LinkList(struct LinkListNode* header);

	//清空链表
	void Clear_LinkList(struct LinkListNode* header);


#ifdef __cplusplus
}
#endif // __cplusplus
~~~

实现文件，不要忘记引用头文件
~~~
#include "LinkList.h"
~~~

~~~
//初始化链表
struct LinkListNode* Init_LinkList() {

	//为头结点分配内存
	struct LinkListNode* header = malloc(sizeof(struct LinkListNode));
	header->data = 0;
	header->next = NULL;

	//定义辅助指针用于指向当前链表的尾部结点
	struct LinkListNode* pRear = header;

	int val = 0;

	while (true) {

		printf("请输入需要插入的数据（输入0退出）\n");
		scanf("%d", &val);

		if (val == 0)
			break;

		struct LinkListNode* newNode = malloc(sizeof(struct LinkListNode));
		newNode->data = val;
		newNode->next = NULL;

		pRear->next = newNode;
		pRear = newNode;

	}

	return header;

}
~~~
其实链表的插入和删除结点操作十分相似，以下为示意图，就是定义两个指针一前一后记录目标元素的前驱结点和后继结点。
![444fcf5f.png](https://miao.su/images/2019/07/17/444fcf5f.png)
~~~
//在值为oldvalue的位置插入一个newvalue
void InsertByValue_LinkList(struct LinkListNode* header, int oldValue, int newValue) {

	if (NULL == header)
		return;

	//两个辅助指针，一前一后用于定位
	struct LinkListNode* prevPoint = header;
	struct LinkListNode* currentPoint = prevPoint->next;

	while (currentPoint != NULL) {

		if (currentPoint->data == oldValue)
			break;

		prevPoint = currentPoint;
		currentPoint = currentPoint->next;

	}

	//从while中跳出有两种情况，一种是currentPoint为NULL即未找到与oldValue，另一种是已经找到了
	if (currentPoint == NULL)
		return;

	struct LinkListNode* newNode = malloc(sizeof(struct LinkListNode));
	
	prevPoint->next = newNode;
	newNode->next = currentPoint;
	newNode->data = newValue;

}
~~~
~~~
//删除值为value的结点
void RemoveByValue_LinkList(struct LinkListNode* header, int delValue) {

	if (NULL == header)
		return;

	struct LinkListNode* prePoint = header;
	struct LinkListNode* currentPoint = header->next;

	while (currentPoint != NULL) {

		if (currentPoint->data == delValue)
			break;

		prePoint = currentPoint;
		currentPoint = currentPoint->next;

	}

	if (NULL == currentPoint)
		return;

	//重新建立删除结点的前驱结点和后继结点关系
	prePoint->next = currentPoint->next;
	//释放被删除节点的空间
	free(currentPoint);
	currentPoint = NULL;

}
~~~
~~~
//遍历链表
void Foreach_LinkList(struct LinkListNode* header) {

	if (NULL == header)
		return;

	struct LinkListNode* pCurrent = header->next;

	while (pCurrent != NULL) {

		printf("%d\n", pCurrent->data);

		pCurrent = pCurrent->next;

	}

}
~~~
~~~
//销毁链表，不保留头结点
void Destroy_LinkList(struct LinkListNode* header) {

	if (NULL == header)
		return;

	// 唯一与清空链表不同的操作，即一个从头开始释放，一个从头的下一个开始释放。
	struct LinkListNode* currentPoint = header;

	while (currentPoint != NULL) {

		//释放结点前，先用一个临时变量保存它的next结点地址
		struct LinkListNode* tempPoint = currentPoint->next;

		//释放当前结点
		free(currentPoint);

		//指向下一个结点
		currentPoint = tempPoint;

	}

}
~~~
~~~
//清空链表, 仅仅保留头结点
void Clear_LinkList(struct LinkListNode* header) {

	if (NULL == header)
		return;

	struct LinkListNode* currentPoint = header->next;

	while (currentPoint != NULL) {

		//释放结点前，先用一个临时变量保存它的next结点地址
		struct LinkListNode* tempPoint = currentPoint->next;

		//释放当前结点
		free(currentPoint);

		//指向下一个结点
		currentPoint = tempPoint;

	}

	//注意
	header = NULL;

}
~~~


PS：实在搞不懂为什么整个寒假我就一直搁那死磕数据结构，头结点头指针，结构体嵌套指针根本搞不懂，还写了一寒假，哭了。为什么不先学基础💀

早上起床后惯例去泡茶，把水壶放上底座然后回来看了几分钟电脑，觉得水烧开后，去厨房端回水壶，倒了些水进余了凉水的特百惠里允温水喝，然后再沏茶，把茶杯盖盖好。看了挺久电脑渴了，端起茶杯正准备惬意的来一口，一看，怎么颜色不是平时的褐色。。。茶叶怎么也没舒展。。。诶怎么杯子这么快就凉了。。。那一刻我才明白烧水没按开关的痛，而且允完我以为的“温水”后当时还喝过一口的，居然这也没反应过来！我是谁？我在哪？我在干嘛？







