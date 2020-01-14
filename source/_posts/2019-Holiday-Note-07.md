---
title: Holiday Note_07
date: 2019-07-19 20:07:00
categories: C
tags: [Note,C,DataStructure]
---

# Holiday Note_07

一周了。今天接触了**动态数组**，挺简单的，主要考虑的应该是每一次增加的空间大小的逻辑。又学习了另外一种结构的**单向链表**，也就是额外用一个链表结构去嵌套结点结构的设计。最重要的学习了**封装**的方式去实现对数据结构的操作，不再返回链表的指针，而是使用万能指针避免使用者进行操作数据结构时对数据进行修改；并且广泛使用**回调函数**增加了数据结构的复用性。随着使用回调函数的次数增多，也逐渐开始体会到了封装所带来的便利了，理解这一部分尤其重要。对于**单向链表各个函数定义的参数那部分一定要读懂其设置的含义**。

<!-- more -->
---

## 动态数组

第一次接触动态数组，听名字DynamicArray感觉很厉害，其实实现起来，和普通的数组结构差不多，主要有三个成员，分别是**数组空间首地址(二维指针)，容量大小，当前大小**，唯一不同的就是判断空间不足时需要再分配一块新空间，再将原空间的数据拷贝过去。

### 动态数组头文件
~~~
struct DynamicArray {

	//数组存储元素的空间的首地址
	void** addr;

	//数组能够在内存中存取的最大元素的个数
	int capacity;

	//当前存储数据的内存中一共有多少个元素
	int size;

};

//初始化数组
struct DynamicArray* Init_DynamicArray(int capacity);

//插入元素
void Insert_DynamicArray(struct DynamicArray* arr, int pos, void* data);

//按位置删除
void DeleteByPos_DynamicArray(struct DynamicArray* arr, int pos);

//按值删除
void DeleteByValue_DynamicArray(struct DynamicArray* arr, void* delData, int(*CompareCallback)(void*, void*));

//遍历数组
void Foreach_DynamicArray(struct DynamicArray* arr, void(*_callback)(void*));

//销毁数组
void Destroy_DynamicArray(struct DynamicArray* arr);
~~~

---

### 动态数组实现文件
#### 初始化数组
~~~
struct DynamicArray* Init_DynamicArray(int capacity) {

	if (capacity <= 0) {
		return NULL;
	}

	struct DynamicArray* arr = malloc(sizeof(struct DynamicArray));

	if (NULL == arr) {
		return NULL;
	}

	arr->addr = malloc(sizeof(void*) * capacity);

	arr->capacity = capacity;

	arr->size = 0;

	return arr;

}
~~~

#### 插入元素(重点，需要判断空间是否足够集进行**分配空间**，**拷贝数据**，**释放原空间**操作)
~~~
void Insert_DynamicArray(struct DynamicArray* arr, int pos, void* data) {

	if (NULL == arr) {
		return;
	}

	if (NULL == data) {
		return;
	}

	//传入的位置不管是小于0还是大于容量，都让插入的位置为最后一个，因为数组元素间不能不连续
	if (pos < 0 || pos >= arr->capacity) {

		pos = arr->size;

	}

	//当存储的数据个数已经等于容量时，需要增长数组空间
	if (arr->size >= arr->capacity) {

		//分配更大的内存空间，暂时不考虑增长策略，默认直接翻倍
		int newCapacity = arr->capacity * 2;
		void** newAddr = malloc(sizeof(void*) * newCapacity);

		//将原空间的数据拷贝至新分配的空间中
		memcpy(newAddr, arr->addr, sizeof(void*) * arr->size);

		//释放原空间内存
		free(arr->addr);

		//更新数组的指向以及容量
		arr->addr = newAddr;
		arr->capacity = newCapacity;
	}

    //为要插入的元素腾出空间
	for (int i = arr->size - 1; i >= pos; --i) {

		arr->addr[i + 1] = arr->addr[i];

	}

	//将元素插入并且更新数组存储数据个数
	arr->addr[pos] = data;
	arr->size++;

}
~~~

#### 按位置删除
功能不够完善，当*删除最后一个元素时*，即pos == size-1的情况时，并未进入循环，而是直接size--，实际该元素仍然存在，在测试文件中，我仍可以将依旧删除的最后一个元素进行打印输出。
~~~
void DeleteByPos_DynamicArray(struct DynamicArray* arr, int pos) {

	if (NULL == arr) {
		return;
	}

	if (pos < 0 || pos > arr->size - 1) {
		return;
	}

	//！！！！！
	//当删除最后一个元素时，即pos == size-1的情况时，并未进入循环，而是直接size--，实际该元素仍然存在
	for (int i = pos; i < arr->size - 1; ++i) {

		arr->addr[i] = arr->addr[i + 1];

	}

	arr->size--;

}
~~~

#### 按值删除
使用了回调函数对数据进行比对操作，传递数据地址和需要删除的值的指针，用户根据不同数据类型编写对应的比对函数。
~~~
void DeleteByValue_DynamicArray(struct DynamicArray* arr, void* delData, int(*CompareCallback)(void*, void*)) {

	if (NULL == arr) {
		return;
	}

	if (NULL == delData) {
		return;
	}

	if (NULL == CompareCallback) {
		return;
	}

	for (int i = 0; i < arr->size; ++i) {

		if (CompareCallback(arr->addr[i], delData)) {

			DeleteByPos_DynamicArray(arr, i);
			break;

		}

	}

}
~~~

#### 遍历数组
同样使用了回调函数
~~~
void Foreach_DynamicArray(struct DynamicArray* arr, void(*_callback)(void*)) {

	if (NULL == arr)
		return;

	for (int i = 0; i < arr->size; ++i) {

		_callback(arr->addr[i]);

	}

}
~~~

#### 销毁数组
~~~
void Destroy_DynamicArray(struct DynamicArray* arr) {

	if (NULL == arr) {
		return;
	}

	if (arr->addr != NULL) {
		free(arr->addr);
		arr->addr = NULL;
	}

	free(arr);
	arr = NULL;

}
~~~

---

### 动态数组测试文件
定义一个结构体数组，调用动态数组进行存储并操作，测试文件需要根据定义的数据类型编写对应回调函数才能调用写好的数据结构操作数据，注意看最后的将删除元素打印步骤。
~~~
struct Person {
	char name[64];
	int age;
};

//遍历回调函数
void myPrint(void* data) {

	struct Person* arrayData = (struct Person*)data;
	printf("name: %s age: %d\n", arrayData->name, arrayData->age);

}

//比较回调函数
int myCompare(void* arrData, void* delData) {

	if (NULL == arrData) {
		return 0;
	}

	if (NULL == delData) {
		return 0;
	}

	struct Person* ArrData = (struct Person*)arrData;
	struct Person* DelData = (struct Person*)delData;

	return (strcmp(ArrData->name, DelData->name) == 0) && (ArrData->age == DelData->age);

}

void test() {

	struct Person person1 = { "aaa",10 };
	struct Person person2 = { "bbb",20 };
	struct Person person3 = { "ccc",30 };
	struct Person person4 = { "ddd",40 };
	struct Person person5 = { "eee",50 };

	struct Person person6 = { "fff",60 };

	// 初始化一个容量只有5的动态数组
	struct DynamicArray * testArray= Init_DynamicArray(5);

	Insert_DynamicArray(testArray, 0, &person1);
	Insert_DynamicArray(testArray, 0, &person2);
	Insert_DynamicArray(testArray, 0, &person3);
	Insert_DynamicArray(testArray, 1, &person4);
	Insert_DynamicArray(testArray, 1, &person5);

	printf("当前数组的容量为 %d \n", testArray->capacity);

	//插入六个数据检测是否实现容量增长
	Insert_DynamicArray(testArray, 666, &person6);

	printf("此时数组的容量为 %d \n", testArray->capacity);

	Foreach_DynamicArray(testArray, myPrint);// 3 5 4 2 1 6

	DeleteByPos_DynamicArray(testArray, 5);

	printf("------按位置删除-----\n");

	Foreach_DynamicArray(testArray,myPrint);// 3 5 4 2 1

	struct Person pTest = { "aaa", 10 };
	DeleteByValue_DynamicArray(testArray, &pTest, myCompare);

	printf("-------按值删除------\n");

	Foreach_DynamicArray(testArray, myPrint);//3 5 4 2


	printf("----\n");

	//按值删除的内部由于调用了按位置删除的函数，因此，对数组当前最后一个元素进行删除时依旧存在没有真正删除的问题
	//因为按位置删除对最后一个元素的删除时，并未采取清除或替换操作，仅仅是进行了size--操作，因此可以通过直接访问该元素地址进行输出
	printf("第五个元素并未删除 name: %s age: %d\n", ((struct Person*)testArray->addr[4])->name, ((struct Person*)testArray->addr[4])->age);
	printf("第六个元素并未删除 name: %s age: %d\n", ((struct Person*)testArray->addr[5])->name, ((struct Person*)testArray->addr[5])->age);

	Destroy_DynamicArray(testArray);

}


int main() {
	test();
}
~~~
测试输入如下：
![699829f16d3528ced1cf85eeff57238906a5.png](https://miao.su/images/2019/07/19/699829f16d3528ced1cf85eeff57238906a5.png)

## 单向链表
与之前写的链表不同的是，这次定义了**两个结构体**，一个是*结点结构*，一个是*链表结构*，其中链表结构嵌套了结点结构，存放了**链表的头结点**，**结点个数**。后期可以根据需要加入类似**尾指针**，让尾插法更加高效的操作链表的一系列成员。

还有一点就是，**封装的思想**。之前写的链表，初始化传递回去的直接就是一个*指向数据结构首地址的数据结构类型指针*，虽然这样可以更加方便操作数据结构，但不加以约束的会带来许多**弊端**：拿到指针后可以随意修改数据，还可以对数据进行清空。

因此这次在实现单向链表时，**初始化数据结构**后传递回去的是一个**万能指针类型**，这样用户在得到该指针后，只能使用这个指针调用我们数据结构已经写好的功能，比如获取结点个数等之前可以直接通过指针操作获取的数据。

在写数据结构的过程中，由于传递的是万能指针类型，所以实现各个功能都需要进行强转万能指针对数据结构进行操作，因为是自己实现数据结构，知道数据结构的类型因此可以通过强转进行操作，而用户则不能进行这种操作。

而且封装的思想还体现在了**插入数据**时接收数据的形参也是万能指针，即我们写数据结构时无需考虑用户传递的数据类型，当实现遍历打印，按值删除这些需要**明确清楚数据类型的功能**时，在函数的形参中使用**函数指针**，这些**回调函数**均由用户去实现，我们的函数指针指向用户写好的回调函数后，根据返回值便可以对数据进行上述功能的实现了。

### 单向链表头文件
**这部分是重点！！！**

增加了使用typedef给万能指针，函数指针重新命名的操作，目的是增加程序的可读性。而且不在头文件中定义数据结构了而是放到了实现文件中，也是封装的思想，**让用户看到更少的细节**，仅需要注重调用功能的实现。
~~~
//使用void*保护链表
	typedef void* LList;
	typedef void(*FOREACH)(void* );
	typedef int(*COMPARE)(void*, void*);

	//初始化链表，不再返回指向链表指针类型
	LList Init_LinkList();

	//插入结点
	void Insert_LinkList(LList list, int pos, void* data);

	//遍历链表
	void ForEach_LinkList(LList list, FOREACH myfoeach);

	//按位置删除结点
	void DeleteByPos_LinkList(LList list, int pos);

	//按值删除结点
	void DeleteByValue_LinkList(LList list, void* data, COMPARE compare);

	//链表大小
	int Size_LinkList(LList list);

	//清空链表
	void Clear_LinkList(LList list);

	//销毁链表
	void Destroy_LinkList(LList list);
~~~

---

### 单向链表实现文件
#### 单向链表数据结构的定义

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


#### 初始化链表，不再返回指向链表指针类型
~~~
LList Init_LinkList() {

	struct LinkList* list = malloc(sizeof(struct LinkList));

	if (NULL == list) {
		return NULL;
	}

	list->header.next = NULL;
	list->header.data = NULL;
	list->size = 0;

	return list;

}
~~~



#### 插入结点(比较重要)
中间有两条语句是插入结点后重新连接链表的操作，因为没画图，就脑子想了想，结果debug了一个半小时，差点崩溃了，所以数据结构画图很重要啊！！
~~~
void Insert_LinkList(LList list, int pos, void* data) {

	if (NULL == list) {
		return;
	}

	if (NULL == data) {
		return;
	}

	//传入的list是void*类型，需要进行强转
	struct LinkList* myList = (struct LinkList*)list;

	if (pos < 0 || pos > myList->size) {

		pos = myList->size;

	}

	//查找插入位置的前一个结点(重点！)
	struct LinkNode* pCurrent = &(myList->header);
	for (int i = 0; i < pos; ++i) {

		pCurrent = pCurrent->next;

	}

	struct LinkNode* newNode = malloc(sizeof(struct LinkNode));
	newNode->next = NULL;
	newNode->data = data;

	// 插入结点，两条语句的先后顺序很关键，画图！！！再写
	newNode->next = pCurrent->next;
	pCurrent->next = newNode;

	myList->size++;

}
~~~

#### 遍历链表

~~~
void ForEach_LinkList(LList list, FOREACH myfoeach) {

	if (NULL == list) {
		return;
	}

	if(NULL == myfoeach) {
		return;
	}

	//传入的list是void*类型，需要进行强转
	struct LinkList* myList = (struct Links*)list;

	struct LinkNode* pCurrent = myList->header.next;
	
	while (pCurrent != NULL) {
		myfoeach(pCurrent->data);
		pCurrent = pCurrent->next;
	}

}
~~~



#### 按位置删除结点
~~~
void DeleteByPos_LinkList(LList list, int pos) {

	if (NULL == list) {
		return;
	}

	struct LinkList* myList = (struct LinkList*)list;

	if (pos < 0 || pos > myList->size) {
		return;
	}

	// 重点理解以下的pCurrent指针和for循环功能
	struct LinkNode* pCurrent = &(myList->header);

	//找到要删除的结点的前一个结点位置
	for (int i = 0; i < pos; ++i) {

		pCurrent = pCurrent->next;

	}

	//将被删除结点的前驱结点和后继结点进行连接
	struct LinkNode* pDel = pCurrent->next;
	pCurrent->next = pDel->next;

	free(pDel);
	pDel = NULL;

	myList->size--;

}
~~~



#### 按值删除结点(比较重要)
~~~
void DeleteByValue_LinkList(LList list, void* data, COMPARE compare) {

	if (NULL == list)
	{
		return;
	}

	if (NULL == data) {
		return;
	}

	if (NULL == compare) {
		return;
	}

	struct LinkList* myList = (struct LinkList*)list;

	//两个辅助指针变量
	struct LinkNode* pPre = &(myList->header);
	struct LinkNode* pCurrent = myList->header.next;

	while (pCurrent != NULL) {
		
		if (compare(pCurrent->data, data)) {
			// 找到要删除的结点
			pPre->next = pCurrent->next;		
			break;
		}
		pPre = pCurrent;
		pCurrent = pCurrent->next;
	}

	//释放被删除的结点
	free(pCurrent);
	pCurrent = NULL;

	myList->size--;

}
~~~

#### 链表大小
~~~
int Size_LinkList(LList list) {

	if (NULL == list) {
		return -1;
	}
	
	struct LinkList* myList = (struct LinkList*)list;

	return myList->size;

}
~~~

#### 清空链表
~~~
void Clear_LinkList(LList list) {

	if (NULL == list) {
		return;
	}

	struct LinkList* myList = (struct LinkList*)list;

	struct LinkNode* pCurrent = myList->header.next;

	while (pCurrent != NULL) {
		
		//先记录下一个结点的地址
		struct LinkNode* pNext = pCurrent->next;
		//释放当前结点的空间
		free(pCurrent);
		pCurrent = pNext;

	}

	myList->header.next = NULL;

	//将链表大小置为0
	myList->size = 0;

}
~~~

#### 销毁链表
~~~
void Destroy_LinkList(LList list) {

	if (NULL == list) {
		return;
	}

	Clear_LinkList(list);

	free(list);
	list = NULL;

}
~~~



#### 我认为比较重要的一个循环
在插入和按位置删除结点都使用了这个循环，用途是找到**需要操作的结点的前一个结点**。
~~~
struct LinkNode* pCurrent = &(myList->header);
for (int i = 0; i < pos; ++i) {

    pCurrent = pCurrent->next;
    
}
~~~

### 单向链表测试文件
~~~
struct Person {
	char name[64];
	int age;
};

//用于对链表遍历的数据进行转换
void myPrint(void* data) {

	struct Person* Data = (struct Person*)data;
	printf("name: %s age: %d\n", Data->name, Data->age);

}

//用于对按值删除的数据进行比较
int myCompare(void* pre, void* cur) {

	struct Person* p1 = (struct Person*)pre;
	struct Person* p2 = (struct Person*)cur;

	return (strcmp(p1->name, p2->name) == 0) && (p1->age == p2->age);

}

void test() {

	struct Person p1 = { "aa",1 };
	struct Person p2 = { "bb",2 };
	struct Person p3 = { "cc",3 };
	struct Person p4 = { "dd",4 };
	struct Person p5 = { "ee",5 };
	struct Person p6 = { "ff",6 };
	struct Person p7 = { "gg",7 };

	//初始化链表
	LList testList = Init_LinkList();

	// 插入数据
	Insert_LinkList(testList, 0, &p1);
	Insert_LinkList(testList, 1, &p2);
	Insert_LinkList(testList, 2, &p3);
	Insert_LinkList(testList, 3, &p4);
	Insert_LinkList(testList, 4, &p5);
	Insert_LinkList(testList, 5, &p6);
	Insert_LinkList(testList, 6, &p7);

	printf("链表大小初始为 %d \n", Size_LinkList(testList));

	ForEach_LinkList(testList, myPrint);
	printf("-----------------\n");

	//按位置删除
	DeleteByPos_LinkList(testList, 1);

	ForEach_LinkList(testList, myPrint);
	printf("-----------------\n");

	//按值删除
	struct Person testDel = { "gg", 7};
	DeleteByValue_LinkList(testList, &testDel, myCompare);


	ForEach_LinkList(testList, myPrint);
	printf("-----------------\n");

	printf("链表大小为 %d \n", Size_LinkList(testList));

	//清空链表
	Clear_LinkList(testList);

	printf("清空后链表大小为 %d \n", Size_LinkList(testList));

	//销毁链表
	Destroy_LinkList(testList);

}

int main() {
	test();
}
~~~
测试输出如下：
![6607952a5f706902e176b84c1cbcca38405c.png](https://miao.su/images/2019/07/19/6607952a5f706902e176b84c1cbcca38405c.png)
PS:一周了，多写一篇周记吧。