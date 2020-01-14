---
title: Holiday Note_17
date: 2019-07-29 22:08:02
categories: C++
tags: [Note,C++]
---
# Holiday Note_17

**类型转换**，**标准I/O流**，**文件读写**，这些都感觉不是很重要，而且记的东西太多了估计明天就忘了。**异常**是今天的一个重点，try，throw，catch。

<!-- more -->

##	类型转换
1.	静态转换 static_cast
-	使用方式  static_cast< 目标类型>（原始数据）
-	**可以进行基础数据类型转换** 
-	父与子类型转换
-	没有父子关系的自定义类型不可以转换
2.	动态转换 
-	使用方式dynamic_cast< 目标类型>（原始数据）
-	**不可以转换基础数据类型**
-	父子之间可以转换(**父转子不可以**；子转父可以；发生多态都可以)
	
3.	常量转换 
-	使用方式const_cast< 目标类型>（原始数据）
-	**不能对非指针或者非引用进行转换**
4.	重新解释转换 
-	使用方法reinterpret_cast < 目标类型>（原始数据）
-	**最不安全**，最鸡肋 不推荐


---

## 异常

异常处理就是处理程序中的错误。

在异常处理过程中，由**问题检测代码**(try)可以**抛出一个对象**(throw)给**问题处理代码**(catch)，通过这个对象的类型和内容，实际上完成了两个部分的通信。

### 异常捕捉方式**严格类型匹配**
~~~
throw 'a';

catch(char){}
~~~



### ...为其他所有类型的异常，通常写在其余类型捕获之后
~~~
catch (...){}
~~~



### 栈解旋(unwinding)
一句话：**异常被抛出后**，**从进入try块起，到异常被抛掷前**，这期间在**栈上**构造的**所有对象**，都会被**自动析构**。析构的顺序与构造的顺序相反，这一过程称为栈的解旋(unwinding)



### 异常接口声明
-	如果想抛出**特定的类型**异常 ，可以利用异常的**接口声明**
-	void func() throw (int) 只能抛出 int类型
-	void func() throw() **不抛出任何类型异常**
-	若函数抛出了接口不允许的异常，将会调用terminate函数中断程序


### 异常变量的生命周期
-	(MyException e)，会多开销一份数据, 调用拷贝构造
-	(MyExcepiton *e)，需要管理delete
-	推荐(MyException &e)


### 异常的多态
-	利用多态来实现如printError同一个接口调用
-	抛出不同的错误对象，提示不同错误

### 系统标准异常
 需要 #incldue <stdexcept>

例如out_of_range, lenger_error等等。接收的类型名即为抛出的异常名
~~~
throw out_of_range（"a123"） 。。。

catch(out_of_range & e)  {
	cout  <<  e.what();
}
~~~

### string与char* 的转换
~~~
string 转 char *   .c_str();
~~~

---

## 标准I/O流

标准I/O对象:cin，cout，cerr，clog

### 标准输入流
标准输入流对象cin一些常见函数

- cin.get 缓冲区中读取一个字符
- cin.get(两个参数) 不读换行符
- cin.getline () 读取换行 并且扔掉
- cin.ignore 忽略 （N） N代表忽略字符数 
- cin.peek 偷窥   偷看1个字符然后放回去
- cin.putback  放回 把字符放回缓冲区
- cin.fail() 看标志位  0正常 1不正常
- cin.clear()重置标志位
- cin.syne() 清空缓冲区

### 标准输出流

#### 流对象的成员函数
-	int number = 99;
-	cout.width(20);
-	cout.fill('*');
-	cout.setf(ios::left); //设置格式  输入内容做对齐
-	cout.unsetf(ios::dec); //卸载十进制
-	cout.setf(ios::hex); //安装16进制
-	cout.setf(ios::showbase); // 强制输出整数基数  0  0x
-	cout.unsetf(ios::hex);
-	cout.setf(ios::oct);
-	cout << number << endl;

#### 控制符

使用控制符方式则需要**iomanip头文件**
~~~
int number = 99;

 cout << setw(20)

<< setfill('~')

<< setiosflags(ios::showbase) //基数
		
<< setiosflags(ios::left) //左对齐

<< hex // 十六进制

<< number

<< endl;
~~~

PS:下了大半天雨的空气才难得清新一会。