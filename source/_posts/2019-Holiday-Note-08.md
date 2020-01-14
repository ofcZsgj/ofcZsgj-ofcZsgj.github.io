---
title: Holiday Note_08
date: 2019-07-20 20:53:34
categories: C
tags: [Note,C,DataStructure]
---

# Holiday Note_08

又学了一种**单向链表**的写法，链表不再存储数据的地址，而是在定义数据类型时，预留一个存放结点的空间。即可以通过数据头部的结点指向将数据连接起来存储为链表结构，以这种方式实现链表的各种方法减少了代码量(不再需要每次分配，释放结点空间)。还学习了**链式栈**和**顺序栈**，链式栈和链表很像，就是插入删除结点的逻辑变成了先进后出，且插入删除结点在头结点方向进行，头结点始终指向栈顶元素。顺序栈的话和数组差不多，都挺简单的。

<!-- more -->

---

## 单向链表

该链表**结点不再将数据存储在链表结点中**，而是在定义数据时预留4字节空间存放链表结点，通过这种方法定义的链表结构操作会更加方便。其实也不一定说一定是4字节，这只是单向链表的写法，因此具体预留多少字节根据定义的结点类型而定。

以下为此单向链表的结构定义
~~~
typedef void(*FOREACH)(void*);
typedef void* LList;

// 链表结点
struct LinkNode {
	struct LinkNode* next;
};

// 链表结构
struct LinkList {
	struct LinkNode header;//头结点
	int size;
};
~~~

对比一下昨天实现的另外一种单向链表的结构定义
~~~
// 结点数据类型
struct LinkNode {
	void* data;
	struct LinkNode* next;
};

// 链表数据类型
struct LinkList {
	struct LinkNode header;
	int size;
};
~~~

因此在使用前者的链表结构时，数据定义也有所变化
~~~
//用户定义的数据类型需要预留空间
struct Person {
	//关键理解预留空间起到的作用，切记这个结点不能使用指针类型
	//若使用指针类型则为4字节空间，而如果结点数据结构中有两个指针则写法错误
	struct LinkNode node;
	//以下才为用户的数据
	char name[64];
	int age;
};
~~~

下面说一下这种链表在**插入**和**删除**结点的优势

这种链表的插入方式与之前实现的链表相比简化了不少，无需再进行对新结点内存空间的分配，因为插入一个新结点，通过改变插入位置的前驱和后继结点数据预留空间中指针的指向即可。

还有一点，使用void* 接收数据和使用struct LinkNode* 接收数据的结果是一样的，都是将数据的前四个预留字节指针进行重新指向但是若选用struct LinkNode* 做参数，用户传递数据时需要额外多一步强转，导致步骤繁琐增加了操作的细节。
~~~
void Insert_LinkList(LList list, int pos, void* data) {

//不同的步骤，将数据转换为结点类型，这样只操作数据首地址后边预留的空间并不会涉及到对数据进行修改
struct LinkNode* newNode = (struct LinkNode*)data;

}
~~~

---

## 顺序栈

顺序栈的定义如下，和数组很相似了。

同样，使用万能指针去存储数据时很方便，不管用户传入什么类型的数据都可以进行指向
~~~
#define MAX 1024
typedef void* SeqStack;	
// 栈的顺序存储结构
struct SequenceStack {
	void* data[MAX];
	int size;
};

// 将数组下标高的一端当作栈顶一端，因为这样插入和删除元素不需要移动数组元素
~~~


对栈进行操作中，入栈，出栈，栈顶元素是十分常用的。
~~~
//初始化栈
SeqStack Init_SequenceStack();

//入栈
void Push_SequenceStack(SeqStack stack, void* data);

//出栈 
void Pop_SequenceStack(SeqStack stack);

//获得栈顶元素
void* Top_SequenceStack(SeqStack stack);

//获得栈的大小
int Size_SequenceStack(SeqStack stack);

//销毁栈
void Destroy_SequenceStack(SeqStack stack);
~~~

入栈操作时，存入数据的代码，存入前需要判断是否空间已满，这不是链表，数据结构的空间大小需要提前定义。
~~~
myStack->data[myStack->size++] = data;
~~~

获得栈顶元素时，注意下标为size-1
~~~
return myStack->data[myStack->size -1 ];
~~~

销毁栈时，free掉的stack虽然是万能指针类型而不是实际栈的数据类型，但是实际上，分配给栈数据结构的空间实际已经被释放，这部分比较深奥，涉及到了编译器的深层次知识，暂时不考虑实现细节。
~~~
free(stack);
stack = NULL;
~~~

**栈没有遍历的操作，因为要想遍历就需要弹出**，因此测试打印栈的所有元素时，需要使用循环，当栈内的元素大于0时执行以下操作：1、获得栈顶元素，并将其强转为自己定义的数据类型结构进行打印其存储的数据；2、**进行出栈操作**
~~~
while (Size_SequenceStack(testStack) > 0) {

	struct Person* person = (struct Person*)Top_SequenceStack(testStack);

	printf("Name: %s Age: %d\n", person->name, person->age);

	Pop_SequenceStack(testStack);
}
~~~

---

## 链式栈

实现的方式为今天学习的新的单向链表方式，不过是在此单向链表的基础上制定了先进后出的规定。

定义与链表一致
~~~
typedef void* LStack;

struct LinkNode {
	struct LinkNode* next;
};

struct LinkStack {
	struct LinkNode head;
	int size;
};	
~~~

定义的函数功能与上方的顺序栈一致，都是那几个功能。

我认为比较重点的是入栈的操作，因为这是链表，需要理清楚**新插入结点后如何让栈顶是链表的第一个结点，且链表头结点始终指向的是栈顶结点这一逻辑**，最好画个图就明白了，有一点点绕，因为之前没有接触过链式栈。

以下为入栈的一些重要步骤
~~~
void Push_LinkStack(LStack stack, void* data) {

struct LinkStack* myStack = (struct LinkStack*)stack;

//重点是以下三条语句，让新插入的每一个结点都在栈顶，且head永远指向栈顶从而达到链式栈的特点。
struct LinkNode* newNode = (struct LinkNode*)data;
newNode->next = myStack->head.next;
myStack->head.next = newNode;

myStack->size++;

}
~~~

理解了入栈，其实出栈逻辑就很清楚了，核心语句如下
~~~
myStack->head.next = myStack->head.next->next;
~~~

PS:眼睛痛死了，眼药水没起作用，怎样才能看一整天电脑眼睛很舒服，我明明一直开着显示器的comfort view模式的。。。不是蓝光的原因么？



