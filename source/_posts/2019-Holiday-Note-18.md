---
title: Holiday Note_18
date: 2019-07-30 20:00:50
categories: C++
tags: [Note,STL,C++]
---
# Holiday Note_18

使用**STL**写代码的确是方便了很多，STL提供的三大组件(**容器**，**算法**，**迭代器**)实现了**数据和操作分离**，数据由容器管理，操作由定制的算法定义，再由迭代器起将其二者连接，使得开发时多关注于实现功能的逻辑。而不用像以前一样，先定义好数据结构，再实现算法等一系列繁琐的步骤了。BUT 不简单啊。

<!-- more -->

---


## string
string管理char*所分配的内存。每一次string的复制，取值都由string类负责维护，不用担心复制越界和取值越界等。

### 构造函数
~~~
string();//创建一个空的字符串 例如: string str;      
string(const string& str);//使用一个string对象初始化另一个string对象
string(const char* s);//使用字符串s初始化
string(int n, char c);//使用n个字符c初始化 
~~~


### 基本赋值操作
相同达到功能使用string类成员.assign没有直接使用构造函数方便。
~~~
string& operator=(const char* s);//char*类型字符串 赋值给当前的字符串
string& operator=(const string &s);//把字符串s赋给当前的字符串
string& operator=(char c);//字符赋值给当前的字符串
string& assign(const char *s);//把字符串s赋给当前的字符串
string& assign(const char *s, int n);//把字符串s的前n个字符赋给当前的字符串
string& assign(const string &s);//把字符串s赋给当前字符串
string& assign(int n, char c);//用n个字符c赋给当前字符串
string& assign(const string &s, int start, int n);//将s从start开始n个字符赋值给字符串
~~~


### 存取字符操作
使用at方式若越界会抛出**out_of_range**的异常，而使用[]越界程序会直接终止。
~~~
char& operator[](int n);//通过[]方式取字符
char& at(int n);//通过at方法获取字符
~~~



### 拼接操作
~~~
string& operator+=(const string& str);//重载+=操作符
string& operator+=(const char* str);//重载+=操作符
string& operator+=(const char c);//重载+=操作符
string& append(const char *s);//把字符串s连接到当前字符串结尾
string& append(const char *s, int n);//把字符串s的前n个字符连接到当前字符串结尾
string& append(const string &s);//同operator+=()
string& append(const string &s, int pos, int n);//把字符串s中从pos开始的n个字符连接到当前字符串结尾
string& append(int n, char c);//在当前字符串结尾添加n个字符c
~~~

### 查找和替换
~~~
int find(const string& str, int pos = 0) const; //查找str第一次出现位置,从pos开始查找
int find(const char* s, int pos = 0) const;  //查找s第一次出现位置,从pos开始查找
int find(const char* s, int pos, int n) const;  //从pos位置查找s的前n个字符第一次位置
int find(const char c, int pos = 0) const;  //查找字符c第一次出现位置
int rfind(const string& str, int pos = npos) const;//查找str最后一次位置,从pos开始查找
int rfind(const char* s, int pos = npos) const;//查找s最后一次出现位置,从pos开始查找
int rfind(const char* s, int pos, int n) const;//从pos查找s的前n个字符最后一次位置
int rfind(const char c, int pos = 0) const; //查找字符c最后一次出现位置
string& replace(int pos, int n, const string& str); //替换从pos开始n个字符为字符串str
string& replace(int pos, int n, const char* s); //替换从pos开始的n个字符为字符串s
~~~


### 比较操作
~~~
/*
compare函数在>时返回 1，<时返回 -1，==时返回 0。
比较区分大小写，比较时参考字典顺序，排越前面的越小。
大写的A比小写的a小。
*/

int compare(const string &s) const;//与字符串s比较
int compare(const char *s) const;//与字符串s比较
~~~

### string子串
~~~
string substr(int pos = 0, int n = npos) const;//返回由pos开始的n个字符组成的字符串
~~~

### 插入和删除操作
~~~
string& insert(int pos, const char* s); //插入字符串
string& insert(int pos, const string& str); //插入字符串
string& insert(int pos, int n, char c);//在指定位置插入n个字符c
string& erase(int pos, int n = npos);//删除从Pos开始的n个字符 
~~~

### string和c-style字符串转换
**const char*到string存在隐式类型转换，但是string不能自动类型转换为C_string。**
~~~
//string 转 char*
string str = "itcast";
const char* cstr = str.c_str();

//char* 转 string 
char* s = "itcast";
string str(s);
~~~

### 大小写转换
~~~
//大写转换
toupper
//小写转换
tolower
~~~

---

## vector
与普通数组不同的是，vector是**动态空间**，随着元素的加入，它的内部机制会**自动扩充空间**以容纳新元素。即把原空间拷贝到一块新空间，且vector提供的是**随机访问迭代器**

[![vectormodel13ef3.md.png](https://miao.su/images/2019/07/30/vectormodel13ef3.md.png)](https://miao.su/image/TPULP)

> 一旦引起空间的重新配置，指向原vector的所有迭代器就都失效了。

-	构造、赋值
-	大小 size 重置大小 resize 容量 capacity
-	是否为空 empty 
-	巧用swap收缩空间 (匿名对象方法)
-	reserve 预留空间
-	insert 插入（迭代器） erase删除 （迭代器） clear（）清空容器
-	pop_back 尾删 front返回第一个数据，back返回最后一个数据
-	逆序遍历 使用reverse_iterator迭代器 (rbegin rend ++)



### 构造
~~~
vector<T> v; //采用模板实现类实现，默认构造函数
vector(v.begin(), v.end());//将v[begin(), end())区间中的元素拷贝给本身。
vector(n, elem);//构造函数将n个elem拷贝给本身。
vector(const vector &vec);//拷贝构造函数。


int arr[] = {2,3,4,1,9};
vector<int> v1(arr, arr + sizeof(arr) / sizeof(int)); 
~~~

### 需要注意的几个API
~~~
resize(int num);//重新指定容器的长度为num，若容器变长，则以默认值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除。
resize(int num, elem);//重新指定容器的长度为num，若容器变长，则以elem值填充新位置。如果容器变短，则末尾超出容器长>度的元素被删除。
capacity();//容器的容量
reserve(int len);//容器预留len个元素长度，预留位置不初始化，元素不可访问。
~~~

### 关于swap的技巧(通过匿名对象收缩空间)
~~~
vector<int> v1;
for (int i = 0; i < 100000; ++i) {
	v1.push_back(i);
}

cout << "容量 " << v1.capacity() <<endl;//138255
cout << "大小 " << v1.size() <<endl;//10000

v1.resize(3);//进行resize后，元素大小变小了，但是容量依旧不变

cout << "容量 " << v1.capacity() << endl;//138255
cout << "大小 " << v1.size() << endl;//3


vector<int>(v1).swap(v1);//利用匿名对象，将v1传给拷贝给匿名对象时不会拷贝空的空间，即根据大小拷贝
//再进行匿名对象和v1的交换，实则交换指针指向，由于匿名对象的容量为v1元素大小，因此达到了收缩空间目的。

cout << "容量 " << v1.capacity() << endl;//3
cout << "大小 " << v1.size() << endl;//3
~~~


### reserve使用技巧
~~~
vector<int> v;

int* p = NULL;//原理：每次空间扩充p指向的便不再是原来vector空间的地址了
int num = 0;//记录容器重新开辟了多少次空间
//在有些已知需要多大空间的情况下，为了提高效率，提前给vector容器分配好大空间减少不断进行动态增加空间的过程
	
v.reserve(100000);//不提前分配时，需要30次

for (int i = 0; i < 100000; ++i) {

	v.push_back(i);

	if (p != &(v[0])) {
		p = &(v[0]);
		++num;
	}

}

cout << num << endl;//30 change 1
~~~

---

### deque

与vector的差别：支持头部删除 头部插入
-	pop_front
-	push_front

> Deque容器和vector容器最大的差异，一在于deque允许使用**常数项**时间对**头端进行元素的插入和删除操作**。二在于deque**没有容量的概念**，因为它是**动态**的以**分段连续空间组合**而成，随时可以增加一段新的空间并**链接**起来，

[![dequemodel9ffa4.md.png](https://miao.su/images/2019/07/30/dequemodel9ffa4.md.png)](https://miao.su/image/TPI6Y)


> Deque采取一块所谓的map(注意，不是STL的map容器)作为**主控**，这里所谓的map是一小块连续的内存空间，其中每一个元素(此处成为一个结点)都是一个指针，指向另一段连续性内存空间，称作缓冲区。**缓冲区**才是deque的存储空间的主体。

[![deque33cfe.md.png](https://miao.su/images/2019/07/30/deque33cfe.md.png)](https://miao.su/image/TP34o)

PS:大学老师整的和小学老师没什么区别，这说明了什么问题:）