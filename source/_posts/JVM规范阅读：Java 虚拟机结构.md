---
title: JVM规范阅读：Java 虚拟机结构
date: 2020-12-04 12:57:50
comments: true
categories:
 - Java
 - JVM
tags: 
 - java
 - jvm
 - 虚拟机
---
## 基本结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200802155454144.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)

---
## 数据类型
分两类：原始类型、引用类型

JVM运行前，编译器会先尽可能的完成类型检查
JVM直接支持对象，实例或数组（数组在JVM里是对象），JVM中用reference类型来表示对某个对象的引用，可以想象成指向对象的指针。

---

## 原始类型与值
### 数值类型
#### 整数类型
java基础数据类型除char外都是有符号的二进制补码整数，默认值都为0。

 1. byte：**8**位，取值范围 [-128 ~ 127]（-2的**7次方** ~ 2的7次方 - 1）
 2. short：**16**位，取值范围 [-32768 ~ 32767]（-2的**15次方** ~ 2的15次方 - 1）
 3. int：**32**位，取值范围 [-2147483648 ~ 2147483647]（-2的**31次方** ~ 2的31次方 - 1）
 4. long：**64**位，取值范围 [-9223372036854775808 ~ 9223372036854775807]（-2的**63次方** ~ 2的63次方 - 1）
 5. **char：16位无符号整数表示默认值为Unicode的null码点（'\u0000'），取值范围 [0 ~ 65535]**

#### 浮点类型

JVM 中的浮点类型按照 **[IEEE 754标准](https://baike.baidu.com/item/IEEE%20754/3869922?fromtitle=IEEE754%E6%A0%87%E5%87%86&fromid=10427270)** 实现

 1. float：32位
 2. double：64位（java 默认）

##### 取值
###### 1. 内存结构
float和double的范围是由指数的位数来决定的。
float的指数位有8位，而double的指数位有11位，分布如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200802105207529.gif)
float：
**1bit（符号位） 8bits（指数位） 23bits（尾数位）**    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200802105324115.gif)
double：
**1bit（符号位） 11bits（指数位） 52bits（尾数位）**
        
于是，float的指数范围为 [-128 ~ 127] ，而double的指数范围为 [-1024 ~ 1023]，并且指数位是按补码的形式来划分的。
其中负指数决定了浮点数所能表达的绝对值最小的非零数；而正指数决定了浮点数所能表达的绝对值最大的数，也即决定了浮点数的取值范围。

**float的范围为 [-2^128  ~  2^127]，也即 [-3.40E+38 ~ 3.40E+38]；
double的范围为 [-2^1024 ~  2^1023]，也即 [-1.79E+308 ~ 1.79E+308]。**

###### 2. 精度
float和double的**精度是由尾数的位数来决定的**。浮点数在内存中是按**科学计数法来存储的**，其整数部分始终是一个隐含着的“1”，由于它是不变的，故不能对精度造成影响。

float：2^23 = 8388608，一共七位，由于最左为1的一位省略了，这意味着最多能表示8位数： 2*8388608 = 16777216 。有8位有效数字，但绝对能保证的为7位，**也即float的精度为7~8位有效数字**；

double：2^52 = 4503599627370496，一共16位，同理，**double的精度为16~17位**。

之所以不能用f1==f2来判断两个数相等，是因为虽然f1和f2在可能是两个不同的数字，但是受到浮点数表示精度的限制，有可能会错误的判断两个数相等！

<font color=red>
所以浮点数只适用于科学计算，不适合普通计算及比较，普通的使用BigDecimal提供的方法进行比较或运算，但要注意在构造BigDecimal的时候使用float、double的字符串形式构建。</font >

### boolean类型

 - true
 - false（默认值）

没有专用的字节码指令，**在JVM 中用 int 数据类型来代替boolean 值。**

true值采用1表示，false值采用0表示。
<br/>

**JVM 支持 boolean 数组，操作指令与 byte数组 共用。**
> Oracle公司虚拟机里用 **bootlean 数组**用 byte数组表示，每个 boolean 元素占 8位。

### returnAddress类型
指向操作码的指针，此操作码与JVM指令相对应。

returnAddress类型会被JVM 的 jsr、ret 和 jsr_w 指令所使用，其指向的是一条虚拟机指令的操作码。

---


## 引用类型与值
JVM 中有三种引用类型：类类型、数组类型、接口类型
这些类型的值分别指向动态创建的类实例、数组实例和实现了某个接口的类实例或数组实例。

### 数组
 - 组件：类型最外面那一维的元素类型叫做数组的**组件类型**（如 int[][][]，其组件类型可以理解为 int[][]）
 - 元素：数组组件类型如果还是数组就不断一直取这个小数组的组件类型，直到其不在为数组，这时这种类型就称为数组的**元素类型**，元素类型必须是**原生类型**、**类类型**或**接口类型**之一。

### null
特殊值，当一个引用不再指向任何对象的时候，它的值就用 null表示。一个为 null的引用，起初并不具备任何实际的运行期类型，但它可以转为任意的引用类型。**引用类型的默认值就是 null。**

---
## 运行时数据区

### pc（program counter） 寄存器
JVM支持多线程，每条JVM线程都有<font color=red>**自己的**</font> pc寄存器。在任意时刻，一条JVM线程只会执行一个方法的代码，这个正在被线程执行的方法称为该线程的当前方法。**如果这个方法不是 native 的，那 pc寄存器就保存 JVM正在执行的字节码指令的地址，如果是 native 方法，那 pc寄存器的值是 undefined。** pc寄存器的容量至少应当能保存一个 returnAddress 类型的数据或者一个与平台相关的本地指针的值。

> 它的作用可以看做是当前线程所执行的字节码的行号指示器。在虚拟机的概念模型里（仅是概念模型，各种虚拟机可能会通过一些更高效的方式去实现），字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。
> 　　由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）只会执行一条线程中的指令。**因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间的计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。**
> 　　如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Natvie方法，这个计数器值则为空（Undefined）。
> 此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。

### <font color=blue>JVM 栈</font>
每一条 JVM线程都有自己<font color=red>**私有的**</font> JVM栈，这个栈与线程同时创建，用于存储栈帧。JVM栈的作用与传统语言（如C语言）中的栈非常类似，**用于存储局部变量与一些尚未计算好的结果**。另外，它在方法调用和返回中也扮演了很重要的角色。因为除了栈帧的出栈和入栈之外，JVM栈不会再受其他的影响，所以栈帧可以在堆中分配，JVM栈所使用的内存不需要保证是连续的。

#### 异常
- 线程申请使用的内存超过JVM栈最大容量，会抛出 **StackOverflowError异常**；
- JVM栈可以动态扩展，如果扩展过程中没有申请到足够的内存或者在创建新的线程时没有足够的内存去创建对应的JVM栈，则会抛出 **OutOfMemoryError 异常**。

### <font color=green>Java 堆</font>
JVM中，堆（heap）是**可供各个线程<font color=red>共享的</font>运行时内存区域，也是供所有类实例和数组对象分配内存的区域**。
在JVM启动时创建，它存储了被GC管理的各种对象，JVM并未假设采用何种技术去实现自动内存管理系统，具体有实现者指定。容量可调，内存不需要保证是连续的。

#### 异常
应用实际所需容量大于GC能提供的最大容量，JVM会抛出 **OutOfMemoryError 异常**。

### <font color=green>方法区</font>
在JVM中，**方法区是可供各个线程<font color=red>共享的</font>运行时内存区域**。方法区与传统语言中的编译代码存储区或者操作系统进程的正文段的作用非常类似，**它存储了每个类的结构信息**，例如，运行时常量池、字段和方法数据、构造函数和普通方法的字节码内容，还包括一些在类、实例、接口初始化时用到的特殊方法。
在JVM启动的时候创建，**<font color=green>是堆的逻辑区</font>**，简单的JVM实现可以选择这区域不实现GC管理。容量可调，内存不需要保证是连续的。

#### 异常
与堆一致。

### <font color=green>运行时常量池</font>
**常量池在<font color=green>方法区</font>中分配**
每个类文件中的常量，运行时可被方法或字段引用。
每个常量池项最多为 **65535**个，由ClassFile结构中16位的 constant_pool_count 字段所决定的。

#### 异常
与方法区一致。

### 本地方法栈
用来支持 native 方法

#### 异常
与方JVM栈一致。

> 本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native方法服务。虚拟机规范中对本地方法栈中的方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（譬如Sun
> HotSpot虚拟机）直接就把本地方法栈和虚拟机栈合二为一。与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。

---
## <font color=blue>栈帧</font>
相当于运数据、方法的输工具

栈帧（frame）是用来存储数据和部分过程结果的数据结构，同时也用来处理动态链接、方法返回值和异常分派。
随着方法调用而创建，随着方法结束而销毁。

栈帧的存储空间由创建它的线程 **<font color=blue>分配在JVM栈中</font>**，每个栈帧都有自己的**本地（局部）变量表**和**操作数栈**和指向当前方法所属类的**运行时常量池**的引用。

本地变量表和操作数栈的容量在编译期确定，并通过相关方法的code属性保存及提供给栈帧使用。因此大小是取决于JVM的实现，实现者可以在调用方法时给它们分配内存。

某个时间点上活跃的那个方法的栈帧称为**当前栈帧**，对应的类称为**当前类**，操作都是对当前栈帧操作的。如果当前方法结束或调用了其他方法，当前栈帧就不再是当前栈帧了，新的栈帧创建，控制权交给新的栈帧。调用方法执行完毕，返回后，JVM丢弃当前栈帧，前一栈帧又重新称为当前栈帧。

### 局部变量表
储存与类或接口中的二进制表示之中，即通过方法的 code属性保存及提供给栈帧使用。

一个局部变量表可以保存 boolean、byte、char、short、int、float、reference或returnAddress类型的数据。两个局部变量可以保存long或double类型数据（取值是从第一个开始的）。

JVM使用局部变量表来完成方法调用时的参数传递。当<font color=red>调用**方法**</font>时，**它的参数将会依次传递到局部变量表中从0开始的连续位置上**。当<font color=red>调用**实例方法**</font>时，**第0个局部变量参数一定用来存储该实例方法所在对象的引用（this）**。后续的其他参数将会传递至局部变量表中**从1开始的连续位置上**。

### 操作数栈
一个后进先出（LIFO）的栈，栈帧中操作数栈的最大深度由编译期决定，并且通过方法的code属性保存及提供给栈帧使用。
操作数栈中的数据必须正确的操作。例如不能入栈两个int类型的数据，然后当做long类型去操作，或者入栈两个float类型数据，然后用iadd对它们求和。**有一小部分指令（dup 和 swap）可以不关注具体数据类型**，把所有在运行时数据区中的数据当作**裸数据类型（raw type）**。这些指令不可修改数据和拆分原本不可拆分的数据。

### 动态链接
动态链接的作用就是**将这些以符号引用所表示的方法转换位实际方法的直接引用**。

类加载的过程中 将要解析尚未解析的符号引用，并且将对变量的访问转化为变量在程度运行时，位于储存结构中的正确偏移量。

---
## 对象的表示
---
## 浮点算法
---
## 特殊方法
默认的构造方法 ` <clinit>` 一个非合法的 Java 方法名，由编译器命名。

## 异常
由当前线程执行的某个操作所导致的，异常称为同步异常。与之相对，异步异常可以在程序中随时发生。
JVM异常出现总是有以下三种原因之一导致的。

- athrow 字节码指令被执行。
- JVM同步检测到程序发生了非正常执行的情况，这时异常必将紧接着在发生非正常执行情况的字节码指令之后抛出，而不会在执行程序的过程中随时抛出。例如：
	- 数组越界
	- 程序在加载或连接时错误 
	- 内存溢出
- 由于以下原因，导致了异步异常：
	- 调用了 Thread或者 ThreadGroup的 stop方法
	- JVM实现发生了内部错误

---
## 字节码指令集简介
简介
### 数据类型与JVM
<table class="table" summary="Type support in the Java Virtual Machine instruction set" border="1">
                        <colgroup>
                           <col>
                           <col>
                           <col>
                           <col>
                           <col>
                           <col>
                           <col>
                           <col>
                           <col>
                        </colgroup>
                        <thead>
                           <tr>
                              <th>操作码</th>
                              <th><code class="literal">byte</code></th>
                              <th><code class="literal">short</code></th>
                              <th><code class="literal">int</code></th>
                              <th><code class="literal">long</code></th>
                              <th><code class="literal">float</code></th>
                              <th><code class="literal">double</code></th>
                              <th><code class="literal">char</code></th>
                              <th><code class="literal">reference</code></th>
                           </tr>
                        </thead>
                        <tbody>
                           <tr>
                              <td><span class="emphasis"><em>Tipush</em></span></td>
                              <td><span class="emphasis"><em>bipush</em></span></td>
                              <td><span class="emphasis"><em>sipush</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tconst</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>iconst</em></span></td>
                              <td><span class="emphasis"><em>lconst</em></span></td>
                              <td><span class="emphasis"><em>fconst</em></span></td>
                              <td><span class="emphasis"><em>dconst</em></span></td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>aconst</em></span></td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tload</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>iload</em></span></td>
                              <td><span class="emphasis"><em>lload</em></span></td>
                              <td><span class="emphasis"><em>fload</em></span></td>
                              <td><span class="emphasis"><em>dload</em></span></td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>aload</em></span></td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tstore</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>istore</em></span></td>
                              <td><span class="emphasis"><em>lstore</em></span></td>
                              <td><span class="emphasis"><em>fstore</em></span></td>
                              <td><span class="emphasis"><em>dstore</em></span></td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>astore</em></span></td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tinc</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>iinc</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Taload</em></span></td>
                              <td><span class="emphasis"><em>baload</em></span></td>
                              <td><span class="emphasis"><em>saload</em></span></td>
                              <td><span class="emphasis"><em>iaload</em></span></td>
                              <td><span class="emphasis"><em>laload</em></span></td>
                              <td><span class="emphasis"><em>faload</em></span></td>
                              <td><span class="emphasis"><em>daload</em></span></td>
                              <td><span class="emphasis"><em>caload</em></span></td>
                              <td><span class="emphasis"><em>aaload</em></span></td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tastore</em></span></td>
                              <td><span class="emphasis"><em>bastore</em></span></td>
                              <td><span class="emphasis"><em>sastore</em></span></td>
                              <td><span class="emphasis"><em>iastore</em></span></td>
                              <td><span class="emphasis"><em>lastore</em></span></td>
                              <td><span class="emphasis"><em>fastore</em></span></td>
                              <td><span class="emphasis"><em>dastore</em></span></td>
                              <td><span class="emphasis"><em>castore</em></span></td>
                              <td><span class="emphasis"><em>aastore</em></span></td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tadd</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>iadd</em></span></td>
                              <td><span class="emphasis"><em>ladd</em></span></td>
                              <td><span class="emphasis"><em>fadd</em></span></td>
                              <td><span class="emphasis"><em>dadd</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tsub</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>isub</em></span></td>
                              <td><span class="emphasis"><em>lsub</em></span></td>
                              <td><span class="emphasis"><em>fsub</em></span></td>
                              <td><span class="emphasis"><em>dsub</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tmul</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>imul</em></span></td>
                              <td><span class="emphasis"><em>lmul</em></span></td>
                              <td><span class="emphasis"><em>fmul</em></span></td>
                              <td><span class="emphasis"><em>dmul</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tdiv</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>idiv</em></span></td>
                              <td><span class="emphasis"><em>ldiv</em></span></td>
                              <td><span class="emphasis"><em>fdiv</em></span></td>
                              <td><span class="emphasis"><em>ddiv</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Trem</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>irem</em></span></td>
                              <td><span class="emphasis"><em>lrem</em></span></td>
                              <td><span class="emphasis"><em>frem</em></span></td>
                              <td><span class="emphasis"><em>drem</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tneg</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>ineg</em></span></td>
                              <td><span class="emphasis"><em>lneg</em></span></td>
                              <td><span class="emphasis"><em>fneg</em></span></td>
                              <td><span class="emphasis"><em>dneg</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tshl</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>ishl</em></span></td>
                              <td><span class="emphasis"><em>lshl</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tshr</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>ishr</em></span></td>
                              <td><span class="emphasis"><em>lshr</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tushr</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>iushr</em></span></td>
                              <td><span class="emphasis"><em>lushr</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tand</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>iand</em></span></td>
                              <td><span class="emphasis"><em>land</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tor</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>ior</em></span></td>
                              <td><span class="emphasis"><em>lor</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Txor</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>ixor</em></span></td>
                              <td><span class="emphasis"><em>lxor</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>i2T</em></span></td>
                              <td><span class="emphasis"><em>i2b</em></span></td>
                              <td><span class="emphasis"><em>i2s</em></span></td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>i2l</em></span></td>
                              <td><span class="emphasis"><em>i2f</em></span></td>
                              <td><span class="emphasis"><em>i2d</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>l2T</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>l2i</em></span></td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>l2f</em></span></td>
                              <td><span class="emphasis"><em>l2d</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>f2T</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>f2i</em></span></td>
                              <td><span class="emphasis"><em>f2l</em></span></td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>f2d</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>d2T</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>d2i</em></span></td>
                              <td><span class="emphasis"><em>d2l</em></span></td>
                              <td><span class="emphasis"><em>d2f</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tcmp</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>lcmp</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tcmpl</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>fcmpl</em></span></td>
                              <td><span class="emphasis"><em>dcmpl</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Tcmpg</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>fcmpg</em></span></td>
                              <td><span class="emphasis"><em>dcmpg</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>if_TcmpOP</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>if_icmpOP</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>if_acmpOP</em></span></td>
                           </tr>
                           <tr>
                              <td><span class="emphasis"><em>Treturn</em></span></td>
                              <td>&nbsp;</td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>ireturn</em></span></td>
                              <td><span class="emphasis"><em>lreturn</em></span></td>
                              <td><span class="emphasis"><em>freturn</em></span></td>
                              <td><span class="emphasis"><em>dreturn</em></span></td>
                              <td>&nbsp;</td>
                              <td><span class="emphasis"><em>areturn</em></span></td>
                           </tr>
                        </tbody>
                     </table>

在 JVM中，实际类型与运算类型直接的映射关系如下表
<table class="table" summary="Actual and Computational types in the Java Virtual Machine" border="1">
                        <colgroup>
                           <col>
                           <col>
                           <col>
                        </colgroup>
                        <thead>
                           <tr>
                              <th>实际类型</th>
                              <th>运算类型</th>
                              <th>分类</th>
                           </tr>
                        </thead>
                        <tbody>
                           <tr>
                              <td><code class="literal">boolean</code></td>
                              <td><code class="literal">int</code></td>
                              <td>1</td>
                           </tr>
                           <tr>
                              <td><code class="literal">byte</code></td>
                              <td><code class="literal">int</code></td>
                              <td>1</td>
                           </tr>
                           <tr>
                              <td><code class="literal">char</code></td>
                              <td><code class="literal">int</code></td>
                              <td>1</td>
                           </tr>
                           <tr>
                              <td><code class="literal">short</code></td>
                              <td><code class="literal">int</code></td>
                              <td>1</td>
                           </tr>
                           <tr>
                              <td><code class="literal">int</code></td>
                              <td><code class="literal">int</code></td>
                              <td>1</td>
                           </tr>
                           <tr>
                              <td><code class="literal">float</code></td>
                              <td><code class="literal">float</code></td>
                              <td>1</td>
                           </tr>
                           <tr>
                              <td><code class="literal">reference</code></td>
                              <td><code class="literal">reference</code></td>
                              <td>1</td>
                           </tr>
                           <tr>
                              <td><code class="literal">returnAddress</code></td>
                              <td><code class="literal">returnAddress</code></td>
                              <td>1</td>
                           </tr>
                           <tr>
                              <td><code class="literal">long</code></td>
                              <td><code class="literal">long</code></td>
                              <td>2</td>
                           </tr>
                           <tr>
                              <td><code class="literal">double</code></td>
                              <td><code class="literal">double</code></td>
                              <td>2</td>
                           </tr>
                        </tbody>
                     </table>

某些对操作数栈进行操作的 Java指令（例如 pop 和 swap 指令）是与具体类型无关的，不过，这些指令必须遵守运算类型分类的限制，这些分类就是上表中的分类。

### 加载和存储指令
本地变量加载到操作数栈: `iload, iload_<n>, lload, lload_<n>, fload, fload_<n>, dload, dload_<n>, aload, aload_<n>`。

将数值从操作数栈存储到局部变量: `istore, istore_<n>, lstore, lstore_<n>, fstore, fstore_<n>, dstore, dstore_<n>, astore, astore_<n>`。

将常量加载到操作数栈: `bipush, sipush, ldc, ldc_w, ldc2_w, aconst_null, iconst_m1, iconst_<i>, lconst_<l>, fconst_<f>, dconst_<d>`。

用于扩充局部变量表的访问索引或立即数的指令: `wide`。

Instructions that access fields of objects and elements of arrays (§2.11.5) also transfer data to and from the operand stack.

Instruction mnemonics shown above with trailing letters between angle brackets (for instance, `iload_<n>`) denote families of instructions (with members `iload_0, iload_1, iload_2, and iload_3 in the case of iload_<n>`). Such families of instructions are specializations of an additional generic instruction (iload) that takes one operand. For the specialized instructions, the operand is implicit and does not need to be stored or fetched. The semantics are otherwise the same (iload_0 means the same thing as iload with the operand 0). The letter between the angle brackets specifies the type of the implicit operand for that family of instructions: `for <n>, a nonnegative integer; for <i>, an int; for <l>, a long; for <f>, a float; and for <d>, a double.` Forms for type int are used in many cases to perform operations on values of type byte, char, and short (§2.11.1).

This notation for instruction families is used throughout this specification.


### 算数指令
- 加法指令: iadd, ladd, fadd, dadd.
- 减法指令: isub, lsub, fsub, dsub.
- 乘法指令: imul, lmul, fmul, dmul.
- 除法指令: idiv, ldiv, fdiv, ddiv.
- 求余指令: irem, lrem, frem, drem.
- 求负值指令: ineg, lneg, fneg, dneg.
- 位移指令: ishl, ishr, iushr, lshl, lshr, lushr.
- 按位或指令: ior, lor.
- 按位与指令: iand, land.
- 按位异或指令: ixor, lxor.
- 局部变量自增指令: iinc.
- 比较指令: dcmpg, dcmpl, fcmpg, fcmpl, lcmp.

### 类型转换指令
从窄到宽，安全转换，从宽到窄，会发生精度丢失问题。
从窄到宽指令：i2l, i2f, i2d, l2f, l2d, and f2d. 
从宽到窄指令：i2b, i2c, i2s, l2i, f2i, f2l, d2i, d2l, and d2f.

### 对象的创建与操作
- 创建类实例指令: new.
- 创建数组指令: newarray, anewarray, multianewarray.
- 访问类字段 (static字段) 和实例字段指令: getstatic, putstatic, getfield, putfield.
- 把一个数组元素加载到操作数栈指令: **b**aload, **c**aload, **s**aload, **i**aload, **l**aload, **f**aload, **d**aload, **a**aload.
- 将一个操作数栈的值存储到数组元素中的指令: **b**astore, **c**astore, **s**astore, **i**astore, **l**astore, **f**astore, **d**astore, **a**astore.
- 获取数组长度的指令: arraylength.
- 检查实例或数组类型的指令: instanceof, checkcast.

### 操作数栈管理指令
- pop, pop2, dup, dup2, dup_x1, dup2_x1, dup_x2, dup2_x2, swap.

### 控制转移指令
- 条件分支: ifeq, ifne, iflt, ifle, ifgt, ifge, ifnull, ifnonnull, if_icmpeq, if_icmpne, if_icmplt, if_icmple, if_icmpgt if_icmpge, if_acmpeq, if_acmpne.
- 复合条件分支: tableswitch, lookupswitch.
- 无条件分支: goto, goto_w, jsr, jsr_w, ret.

JVM中有专门的条件分支指令集来处理 int 和 reference 类型的比较操作，也有专门来检测 null值的指令。

boolean、byte、char和short类型的条件分支比较操作都使用int类型的比较指令来完成，而对于 long、float和double类型的则会先执行对应的比较运算指令，运算指令会返回一个整型数值到操作数栈中随后再执行int类型的比较。

所有的int类型的条件分支比较都是无符号的比较操作。

### 方法调用和返回指令

### 异常抛出
- athrow

### 同步
- monitorenter
- monitorexit

---
## 类库
---
## 共有设计、私有实现
---

## 参考：
- [The Java® Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html#jvms-2.11)
- [Java 浮点数 float和double类型的表示范围和精度](https://blog.csdn.net/zq602316498/article/details/41148063)
- [JVM学习笔记——JVM启动流程和基本结构](https://blog.csdn.net/lyhkmm/article/details/79609542)
