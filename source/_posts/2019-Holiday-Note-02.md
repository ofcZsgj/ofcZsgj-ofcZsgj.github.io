---
title: Holiday Note_02
date: 2019-07-14 19:23:03
categories: C
tags: [Note,C]
---

# Holiday Note_02

放假第二天。今天学的主要只有两块内容：分别是**字符串相关**(字符串格式化函数使用，字符串的拷贝和反转实现方法，查找字串的方法)；**指针作为函数参数时具有的输入输出特性**相关(指针间接赋值，主调函数分配内存与被调函数分配的内存的使用区别，二级指针对指定文件分配，释放空间练习)。难点主要在于理解指针作为函数参数时需要使用哪种类型接收，指针的间接赋值时二级指针的使用，查找子串功能实现时各个指针实现的功能。

<!-- more -->

---

## 字符串相关
以下为sprintf函数以及sscanf函数在格式化字符串时的一些注意要点以及用法场景。
### sprintf 函数
sprintf简单来讲就是按照指定格式，将字符串进行格式化后放入char * str中，与printf相比不过是多了一个接收存放格式化后字符串的指针。具体函数参数以及返回值如下所示。

~~~
int sprintf(char* str, const char* format, ...);
功能：
	根据参数format字符串来转换并格式化数据，然后将结果输出到str指定的空间中，直到出现字符串结束符 '\0'为止。
参数：
	str：字符串首地址
	format：字符串格式，用法和printf()一样
返回值：
	成功：实际格式化的字符个数
	失败： - 1
~~~

这是sprintf的三种常见用法

#### 格式化字符串
~~~
char buf[1024] = { 0 };
sprintf(buf, "Hello %s!", "World");
printf("%s\n", buf);//输出结果为：Hello World!
~~~

#### 拼接字符串
~~~
char* s1 = "Hello";
char* s2 = "World!";
sprintf(buf, "%s %s", s1, s2);
printf("%s\n", buf);//输出结果为：Hello World!
~~~

#### 数字转换为字符串
~~~
int number = 666;
sprintf(buf, "%d", number);
printf("%s\n", buf);//输出结果为666
~~~


---


### sscanf 函数
sscanf函数的主要功能为从指定的字符串中按照给定的格式读取并转换出来。如果说sprintf是放进去，那么sscanf则为取出来，且均按照指定格式进行放入和取出。

~~~
int sscanf(const char* str, const char* format, ...);
功能：
	从str指定的字符串读取数据，并根据参数format字符串来转换并格式化数据。
参数：
	str：指定的字符串首地址
	format：字符串格式，用法和scanf()一样
返回值：
	成功：实际读取的字符个数
	失败： - 1
~~~


以下为sscanf的**六种**常见格式与作用

#### 1、%*s或%*d	
~~~
char* str = "123456abcdefg";
char buf[1024] = { 0 };
sscanf(str, "%*d%s", buf);
printf("buf: %s\n", buf);//输出结果为buf: abcdefg
~~~

**注意**忽略字符串时是忽略到空格或者'\t'
~~~
char* str1 = "abcdef\t123456";
char buf1[1024] = { 0 };
sscanf(str1, "%*s%s", buf1);//此使输出结果为buf: 123456
printf("buf1: %s\n", buf1);//若str1字母与数字间无空格或'\t'，将会把str1全部忽略掉，这种情况输出将为buf1:
~~~

#### 2、%[width]s
~~~
char* str = "123456abcdefg";
char buf[1024] = { 0 };
sscanf(str, "%7s", buf);
printf("buf: %s\n", buf);//输出结果为buf: 123456a
~~~

以下四种特点均为只要与要求不符合立即不进行匹配。简单说就是，不是将整个字符串中匹配的筛选出来，而是从字符串头部开始，判断一个字符格式不符合的即退出或者至'\0'退出

#### 3、%[a-z]
~~~
char* str = "123456abcdefg";
char buf[1024] = { 0 };
sscanf(str, "%*d%[a-c]", buf);//若不加忽略%*d的话，由于第一个字符为'1'，将直接退出，不拷贝
printf("buf: %s\n", buf);//输出结果为buf: abc
~~~

#### 4、%[aBc]
~~~
char* str = "aABb";
char buf[1024] = { 0 };
sscanf(str, "%[aAb]", buf);//从第一个开始比对，若不同依旧是直接不拷贝
printf("buf: %s\n", buf);//输出结果为buf: aA
~~~

#### 5、%[^a]
~~~
char* str = "aABb";
char buf[1024] = { 0 };
sscanf(str, "%[^B]", buf);//从第一个开始比对，若不同依旧是直接不拷贝
printf("buf: %s\n", buf);//输出结果为buf:aA
~~~

#### 6、%[^a-z]
~~~
char* str = "cdaABb";
char buf[1024] = { 0 };
sscanf(str, "%[^a-b]", buf);//从第一个开始比对，若不同依旧是直接不拷贝
printf("buf: %s\n", buf);//输出结果为buf:cd
~~~

#### 综合使用演示
~~~
//取出12uiop
char* str = "abSi#12uiop@Tsi(";
char buf[1024] = {0};
//sscanf(str, "%*[^#]", buf);//输出结果buf:
//sscanf(str, "%*[^#]%s", buf);//输出结果buf:#12uiop@Tsi(
sscanf(str, "%*[^#]#%[^@]", buf);//输出结果:buf: 12uiop
printf("buf: %s\n", buf);
~~~

***

### 字符串与字符数组的易错点
*最重要记住一句话：字符串是以0或者'\0'结尾的字符数组，(数字0和字符'\0'等价)*

#### 字符数组初始化
~~~
//字符数组只能初始化5个字符，当输出的时候，从开始位置 直到找到0结束
char str1[] = { 'h','e','l','l','o',};
printf("str1 = %s", str1);//输出结果将为 str1 = hello烫烫烫剃*堺?

//如果以字符串初始化，那么编译器默认会在字符串尾部添加'\0'
char str2[] = "hello";
~~~


#### sizeof与strlen差别
~~~
//sizeof计算数组大小，数组包含'\0'字符
//strlen计算字符串的长度，到'\0'结束
printf("sizeof str:%d\n", sizeof(str2));//结果为6
printf("strlen str:%d\n", strlen(str2));//结果为5
~~~

---

### 实现字符串拷贝函数的三种写法
三种方法由易到难，由数组到指针，其实指针理解了很简单。

主调用函数为
~~~
test05() {
	char* source = "Hello World";
	char buf[1024];
	copyString01(buf, source);
	copyString02(buf, source);
	copyString03(buf, source);
	printf("%s\n", buf);
}
~~~

#### 方法1
~~~
void copyString01(char*dest,const char*source) {
	int len = strlen(source);
	for (int i = 0; i < len; ++i) {
		dest[i] = source[i];
	}
	dest[len] = '\0';
}
~~~

#### 方法2
~~~
void copyString02(char* dest, const char* source) {
	while (*source) {
		*dest = *source;
		++dest;
		++source;
	}
	*dest = '\0';
}
~~~

#### 我认为最好的方法3
此写法，简洁优雅。
~~~
void copyString03(char* dest, const char* source) {
	while (*(dest++) = *(source++));
}
~~~

***

### 指针和数组方式实现字符串反转
两种写法大同小异，思路均为两个变量，头尾互换即可，关键点在于反转后的字符串结尾进行 '\0' 操作。
#### 数组实现
~~~
void reverseString01(char *str) {
	if (NULL == str)
		return;
	int len = strlen(str);
	//以下为数组方式实现
	int start = 0;
	int end = len - 1;
	while (start < end) {
		char temp;
		temp = str[start];
		str[start] = str[end];
		str[end] = temp;
		++start;
		--end;
	}
	str[len] = '\0';
}
~~~


#### 指针实现
~~~
if (NULL == str)
		return;
	int len = strlen(str);	

	//以下为指针方式实现
	char* pStart = str;
	char* pEnd = str + len - 1;
	while (pStart < pEnd) {
		char temp;
		temp = *pStart;
		*pStart = *pEnd;
		*pEnd = temp;

		++pStart;
		--pEnd;
	}
	*(str + len) = '\0';
~~~

***

### 实现查找子串
该功能理解稍有难度，主要是理清楚逻辑。例如字符串为"abdecdfw","df",首先思考的是找到字符串与字串第一个字符是否吻合，且定义一个指针记录哪一个字符与字串开始吻合，当找到了吻合的字符 'd' 后，关键步骤为，定义两个指针，一个指向字符串此时的位置，一个指向子串第一个字符的位置，分别进行遍历比较，应设定有两种退出情况，一种是比较字符不相等，另一种指向字串的指针指向'\0'返回。即退出后验证若指向字串的字符串已经指向 '\0' 即找到真正字串，此使返回最初用于记录字符串与字串字符开始吻合时位置的指针即可。 

![b89941.png](https://miao.su/images/2019/07/14/b89941.png)

~~~
const char* myStrStr(const char* str, const char* sub) {
	
	const char* mystr = str;
	const char* mysub = sub;

	while (*mystr != '\0') {

		if (*mystr != *mysub) {
			++mystr;
			continue;
		}

		const char* temp_str = mystr;
		const char* temp_sub = mysub;

		while (*temp_sub !='\0') {//第一个字符相等，继续向后比对
			if (*temp_str != *temp_sub) {
				++mystr;
				break;
			}
			++temp_str;
			++temp_sub;
		}

		if (*temp_sub == '\0') {// 匹配成功
			return mystr;
		}
	}
	return NULL;
}

void test() {
	char* str = "abdecdfw";
	char* sub = "df";
	const char* p = myStrStr(str, sub);
	printf("%s\n", p);
}

int main() {
	test();
}
~~~

***

## 指针作为函数参数时具有的输入输出特性

指针的**间接赋值**需要理解的十分到位！
### 指针间接赋值
- 用1级指针形参，去间接修改了0级指针(实参)的值。
- 用2级指针形参，去间接修改了1级指针(实参)的值。
- 用3级指针形参，去间接修改了2级指针(实参)的值。
- 用n级指针形参，去间接修改了n-1级指针(实参)的值。

---

### 指针作为函数参数时具有的输入特性
内容不难，但关键在于理解函数作为参数时的内存分配使用模型。

#### *输入特性*：主调函数分配内存，被调函数使用内存使用内存
~~~
void printString(const char* str) {
	printf("s= %s\n", str);
}

void test01() {
	//主调函数于堆上分配内存
	char* s = malloc(sizeof(char) * 100);
	memset(s, 0, 100);
	strcpy(s, "Hello World\n");
	printString(s);
}
~~~

~~~
void printStringArray(char** arr,int len) {
    // 因arr[0]是char*, 因此需要用char**来接收
	for (int i = 0; i < len; ++i) {
		//arr[i] 等价于 *(arr + i)
		printf("%s\n", arr[i]);
	}
}

void test02() {
	//主调函数于栈上分配内存
	char* arr[] = {
		"aaaaa",
		"bbbbb",
		"ccccc",
		"ddddd",
	};
	//数组名作为函数参数进行传递时，会退化为指向数组首元素类型的指针
	printStringArray(arr, sizeof(arr) / sizeof(arr[0]));
}
~~~


#### *输出特性*：被调函数分配内存，主调函数使用内存使用内存
输出特性即被调函数开辟空间，即若主调函数想使用该空间需要使用指针的间接赋值，如不能成功将被调函数开辟的内存空间地址保存，将可能导致内存泄露，这是重难点。
~~~
void allocateSpace(char **p) {
	//被调函数于堆空间开辟内存
	char* temp = malloc(100);
	memset(temp, 0, 100);

	//指针的间接赋值
	*p = temp;
}

test03() {
	//主调函数使用被调函数开辟的堆空间内存
	char* p = NULL;
	allocateSpace(&p);	
}
~~~

***

### 二级指针针对文件的开辟释放内存练习 (有一定难度的综合练习)
该练习为：读取一个文件，根据其行数与每一行的字符数，为每一行开辟一个堆空间，且每一行的堆空间大小即为该行等到字符数，实现较为繁琐，有很多细节如指针为空需要即使进行返回，文件指针的重置操作，重置缓冲区，关闭文件，释放内存的顺序以及将指针置空等。

获取文件行数模块
~~~
//获取文件行数
int getFileLines(FILE* file) {
	if (NULL == file)
		return -1;

	int lines = 0;
	char buf[1024] = {0};

	while (fgets(buf, 1024, file) != NULL) {
		++lines;
	}

	//注意：恢复文件指针指向初始位置！！！
	fseek(file, 0, SEEK_SET);

	return lines;
}
~~~
读取文件数据并分配空间模块(重点！)
~~~
//读取文件数据，为每一行分配对应大小的堆空间
void readFileData(FILE* file,char** pContents,int fileLines) {

	if (NULL == file)
		return;

	if (NULL == pContents)
		return;

	if (fileLines <= 0) 
		return;
	

	char buf[1024] = { 0 };
	int index = 0;

	while (fgets(buf, 1024, file) != NULL) {

		//计算当前行的字符数
		int curLineLen = strlen(buf) + 1;

		//给当前行开辟对应大小的堆空间
		char* lineContent = malloc(sizeof(char) * curLineLen);

		//将该行数据拷贝进堆空间中
		strcpy(lineContent, buf);

		//重置buf数据
		memset(buf, 0, 1024);

		//指向开辟的空间
		pContents[index++] = lineContent;
	}

	//关闭文件
	fclose(file);
	file = NULL;
}
~~~
打印文件内容模块
~~~
void printData(char** pContents, int fileLines) {
	if (NULL == pContents)
		return;

	if (fileLines <= 0)
		return;

	for (int i = 0; i < fileLines; ++i) {
		printf("第 %d 行：%s", i + 1, pContents[i]);
	}

}
~~~
释放内存空间模块
~~~
void freeSpace(char** pContents, int fileLines) {
	//从里到外释放，与开辟空间顺序相反
	for (int i = 0; i < fileLines; ++i) {
		if (pContents[i] != NULL) {
			free(pContents[i]);
			pContents[i] = NULL;
		}	
	}

	if (pContents != NULL) {
		free(pContents);
		pContents = NULL;
	}
	
}
~~~
以下为函数主体逻辑函数和主函数
~~~
void test() {
	FILE* file = fopen("./test.txt", "r");
	if (!file)//文件打开失败另一种写法
		return;

	int fileLines = getFileLines(file);	
	//printf("行数为：%d\n", fileLines);

	char** pContents = malloc(sizeof(char*) * fileLines);

	//读取文件并对应每行开辟堆空间
	readFileData(file, pContents, fileLines);

	//打印文件内容
	printData(pContents, fileLines);

	//释放堆空间
	freeSpace(pContents, fileLines);
}

int main() {

	test();

}
~~~

***

PS：今天只花了俩小时就总结完了，总体来说这样复习的效果还是不错的，而且越来越熟练，白天码代码时就有意识的写注释进行整理了。但是排版还是个问题，一直觉得排版巨丑，不知道什么时候能好好学学解决下。