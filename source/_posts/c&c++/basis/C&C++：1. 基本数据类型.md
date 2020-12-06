---
title: C&C++：1. 基本数据类型
date: 2020-12-06 17:00:00
comments: true
categories:
 - C&C++
 - 基础
tags: 
 - c
 - c++
---

## 基础不牢，地动山摇！
基本数据类型是所有编程语言的第一门课，必须要好好学习掌握，要非常熟悉每种类型的用途、范围大小、所占内存。

## 位、字节和字
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201206112313525.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
**位（bit）是最小存储单元，可以存储0或1**。
**字节（byte）是常用的计算机存储单位**。几乎对所有机器，1字节均为8位。
**字（word）是设计计算机时给定的自然存储单位**，对于8位的微型计算机，1个字长只有8位。从那以后，个人计算机字增长至16位、32位直到目前的64位。计算机的字长越大，其数据转移越快，允许的内存访问也更多。


> int类型比较特殊，具体的字节数同机器字长和编译器有关。如果要保证移植性，尽量用__int16 __int32 __int64吧
> 
> __int16、__int32这种数据类型在所有平台下都分配相同的字节。所以在移植上不存在问题。
> 
> <font color=red>所谓的不可移植是指：在一个平台上编写的代码无法拿到另一个平台上运行时，不能达到期望的运行结果。</font>
> 
> 例如：**在32为平台上（所谓32位平台是指通用寄存器的数据宽度是32）编写代码，int 类型分配4个字节，而在16位平台是则分配2个字节，那么在16位上编译出来的exe，
其中是为int分配2字节，而在32位平台上运行时，会按照4个字节来解析，显然会出错误的！！**
> 
> 而对于非int行，目前为止，所有的类型分配的字节数都是兼容的，即不同平台对于同一个类型分配相同的字节数！！
> 
> <font color=red>建议：在代码中尽量避免使用int类型，根据不同的需要可以用short,long,unsigned int 等代替。</font>

下面是各个类型一览表【转】

64位指的是cpu通用寄存器的数据宽度是64位的。

|数据类型名称|	字节|	别名|	取值范围|
|:-- |:-- |:-- |:-- |
|int|	*	|signed,signed int|	由操作系统决定，即与操作系统的 "字长" 有关|
|unsigned int|	*	|unsigned	|由操作系统决定，即与操作系统的 "字长" 有关|
|__int8|	1|	char,signed char|	–128 到 127|
|__int16|	2|	short,short int,signed short int|	–32,768 到 32,767|
|__int32|	4	|signed,signed int|	–2,147,483,648 到 2,147,483,647|
|__int64|	8	|无|	–9,223,372,036,854,775,808 到 9,223,372,036,854,775,807|
|bool|	1	|无	|false 或 true|
|char	|1|	signed char|	–128 到 127|
|unsigned char	|1	|无	|0 到 255|
|short|	2|	short int,signed short int|	–32,768 到 32,767|
|unsigned short	|2	|unsigned short int	|0 到 65,535|
|long|	4	|long int,signed long int|	–2,147,483,648 到 2,147,483,647|
|long long	|8|	none (but equivalent to __int64)|	–9,223,372,036,854,775,808 到 9,223,372,036,854,775,807|
|unsigned long	|4|	unsigned long int|	0 到 4,294,967,295|
|enum|	*	|无|	由操作系统决定，即与操作系统的 "字长" 有关|
|float	|4	|无|	3.4E +/- 38 (7 digits)|
|double|	8|	无|	1.7E +/- 308 (15 digits)|
|long double|	8	|无	|1.7E +/- 308 (15 digits)|
|wchar_t|	2	|__wchar_t	|0 到 65,535|


一些与平台无关的整型

|数据类型| 别名	| 符号 | 字节 | 最小值 | 最大值|
|:-- |:--  |:-- |:-- |:-- |:-- |
|int8_t|	signed char|	Signed|	1|	-128|	127|
|uint8_t|	unsigned char|	Unsigned|	1|	0	|255|
|int16_t| short|	Signed|	2|	-32,768|	32,767|
|uint16_t|	unsigned short|	Unsigned|	2|	0	|65,535|
|int32_t|	int|	Signe|	4	|-2,147,483,648|	2,147,483,647|
|uint32_t|	unsigned int|	Unsigned|	4|	0	|4,294,967,295|
|int64_t|	long long|	Signed	|	8	|-9,223,372,036,854,775,808	|9,223,372,036,854,775,807|
|uint64_t|	unsigned long long|	Unsigned|	8	|0	|18,446,744,073,709,551,615|

上面是一些与平台无关的数据类型，由于在32位机器和64位机器中，long占据不同的字节数，所以推荐使用上面的类。

**在32位编译器中，int和long都是占4个字节。**

一般来说，signed 和 unsigned 可看作是修饰 short 和 int 这些类型名词的形容词，不过单独使用 signed 和 unsigned 时，默认了它们的中心词是 int。

## 变量名
**命名规则**
 - 只能用字母字符、数字和下划线（_）。
 - 第一个字符不能是数字。
 - 字符区分大小写。
 - 不能使用关键字。
 - 以两个下划线或下划线和大写字母打头的名称被保留给实现（编译器及其使用的资源）使用。以一个下划线开头的名称被保留给实现，用作全局标识符。
 - C++ 对名称的长度没有限制，但有些平台有限制。

## 整型 short、int、long 和 long long
short 至少 16位；
int 至少与 short 一样长；
long 至少 32位，且至少与 int 一样长；
long long 至少 64位，且至少与 long 一样长。

### 1. 运算符 sizeof 和头文件 limits
- 使用 sizeof 运算符可以得到某类型、变量或表达式结果所占用的字节数。
- 头文件 climits 文件定义了符号常量来表示类型的限制。

### 2. 初始化

```cpp
int test = 20;

// 不推荐
int test1;
test1 = 30;
```

注意：如果不对函数内部定义的变量赋值，该变量的值将是不确定的。这意味着该变量的值将是在它创建之前，相应内存单元保存的值。

**C++ 11 的初始化方式**

大括号初始化（一致性初始化），是非窄化初始化。

```cpp
int emus{7};
int rheas = {12};

int rocs = {};
int psychics{};
```

```cpp
int i1 = 3.14     // 编译器执行了窄化操作（风险）
int i1n = {3.14}  // 窄化错误，小数部分丢失
float f1 = {3.14} // 正确
```

## 无符号类型
注意：unsigned 本身就是 unsigned int 的缩写。

## C++ 如何确定常量的类型
默认存储为 int 型，u 或 U 表示 unsigned int，long 后缀 l 或 L，unsigned 就 ul 或 UL。
依此类推，ll、ull

## char 类型：字符和小整数
字面值，用单引号括起来的单字符。

**char 默认既不是有符号也不是无符号，是否有符号有C++ 实现决定，这样是为了灵活与硬件属性匹配。**

signed char  0 ~ 255

unsigned char -128 ~ 127

存储标准 ASCII 字符有无符号都行。

**wchat_t 宽字符类型，用于处理用一个8位字节无法表示的字符**，如日文汉字系统。是一种整型，与另一种整型（底层（underlying）类型）的长度和符号属性相同。与底层实现相关。

**C++ 11 新增：char16_t 和 char32_t**
都是无符号char 一个占16位（UTF-16），一个占32位（UTF-32），前缀 u 和 U 表示。此类型的出现与 Unicode有关，wchat_t 不够用，如字符串编码、表情符等。

## bool 类型
- true
- false

原则上只需占用1位

提升到 int 1=true、0=false，此外任何数字或指针都可以隐式转换为 bool 任何非零的为 true，零为 false。

## 浮点数
C 和 C++ 规定，float 至少 32位，double 至少48位，且不少于 float，long double 至少和 double 一样多。
通常 float 32位，double   64位，long double 80、96或128位。另外这三种的指数范围至少是 -37到37。

## const 限定符
表示常量，C 中会用 #define 定义符号常量，C++ 中请使用 const。

## auto 关键字
编译器会自动推断类型。

```cpp
auto m {10} // int
auto n {2000L} // long
auto i {2000UL} // unsigned long
auto pi {3.1415926} // double
```
**如果编译器不是最新的，不推荐使用 auto。**
