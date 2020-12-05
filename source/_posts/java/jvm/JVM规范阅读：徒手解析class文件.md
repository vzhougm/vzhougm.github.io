---
title: JVM规范阅读：徒手解析class文件
date: 2020-12-04 12:59:53
comments: true
categories:
 - Java
 - JVM
tags: 
 - java
 - jvm
 - 虚拟机
---
## Class文件结构

```c
ClassFile {
    u4             magic; // 魔数
    u2             minor_version; // 副版本号
    u2             major_version; // 主版本号
    u2             constant_pool_count; // 常量池计数器
    cp_info        constant_pool[constant_pool_count-1]; // 常量池数据区
    u2             access_flags; // 访问标志
    u2             this_class; // 类索引
    u2             super_class; // 父类索引
    u2             interfaces_count; // 接口计数器
    u2             interfaces[interfaces_count]; // 接口表
    u2             fields_count; // 字段计数器
    field_info     fields[fields_count]; // 字段表
    u2             methods_count; // 方法计数器
    method_info    methods[methods_count]; // 方法表
    u2             attributes_count; // 属性计数器
    attribute_info attributes[attributes_count]; // 属性表
}
```
## Java 源码
```java
/**
 * @author Rubin
 * @version v1 2020/7/30 11:05
 */
public class Main {

    double doubleLocals(double d1, double d2) {
        return d1 + d2;
    }

}
```
##  javap 查看字节码

javap -c .\Main.class
```cpp
Compiled from "Main.java"
public class Main {
  public Main();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  double doubleLocals(double, double);
    Code:
       0: dload_1
       1: dload_3
       2: dadd
       3: dreturn
}
```

javap -verbose .\Main.class
```cpp
Classfile /D:/Java/IdeaProjects/ieacs/target/classes/Main.class
  Last modified 2020年8月4日; size 388 bytes
  MD5 checksum 9dbf204007d5615fb43c9c0e03b1d1f5
  Compiled from "Main.java"
class Main
  minor version: 0
  major version: 52
  flags: (0x0020) ACC_SUPER
  this_class: #2                          // Main
  super_class: #3                         // java/lang/Object
  interfaces: 0, fields: 0, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #3.#19         // java/lang/Object."<init>":()V
   #2 = Class              #20            // Main
   #3 = Class              #21            // java/lang/Object
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               LocalVariableTable
   #9 = Utf8               this
  #10 = Utf8               LMain;
  #11 = Utf8               doubleLocals
  #12 = Utf8               (DD)D
  #13 = Utf8               d1
  #14 = Utf8               D
  #15 = Utf8               d2
  #16 = Utf8               MethodParameters
  #17 = Utf8               SourceFile
  #18 = Utf8               Main.java
  #19 = NameAndType        #4:#5          // "<init>":()V
  #20 = Utf8               Main
  #21 = Utf8               java/lang/Object
{
  Main();
    descriptor: ()V
    flags: (0x0000)
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 5: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   LMain;

  double doubleLocals(double, double);
    descriptor: (DD)D
    flags: (0x0000)
    Code:
      stack=4, locals=5, args_size=3
         0: dload_1
         1: dload_3
         2: dadd
         3: dreturn
      LineNumberTable:
        line 8: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       4     0  this   LMain;
            0       4     1    d1   D
            0       4     3    d2   D
    MethodParameters:
      Name                           Flags
      d1
      d2
}
SourceFile: "Main.java"
```

## 解析ing...
### 出现的指令
指令      | 意思  | 码
:-------- | :-----| :--
aload_0  | 从局部变量加载一个 reference 类型值到操作数栈  |  25（0x19）
invokespecial #1   | 调用实例方法，专门用来调用父类方法、私有方法和实例初始化方法  |  183（0xb7）
return  |  方法中返回 void |  177（0xb1）
dload_1  | 从局部变量加载一个 double 类型值到操作数栈中  |  24（0x18）
dload_3  | 从局部变量加载一个 double 类型值到操作数栈中  |  24（0x18）
dadd  |  double 类型数据相加 |  99（0x63）
dreturn  | 从方法中返回一个 double 类型数据  |  175（0xaf）

### Class 文件解析

### 常量池结构
```cpp
cp_info {
    u1 tag;
    u1 info[];
}
```
常量池只包含本身类的常量，不包含父类。最大数量 65,535

**常量池的常量分两类，一是有所属类或接口的，如类的方法、字段等。另一类是没有所属类或接口的（CONSTANT_NameAndType_info），如构造方法。**

类目| 包含
:---|:---
类| 1. **Object类**<br/> 2. 自己本身
字段|1. 本身的字段
方法|1. **默认的构造方法**<br/> 2. 本身的方法
CONSTANT_NameAndType_info| `<init>`



 access_flags
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200804144533503.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
### 涉及到的结构体

```c
CONSTANT_Methodref_info {
      u1 tag（10 0xa）;
      u2 class_index;
      u2 name_and_type_index;
  }

  CONSTANT_Class_info {
      u1 tag（7）;
      u2 name_index;
  }

  CONSTANT_Utf8_info {
      u1 tag（1）;
      u2 length;
      u1 bytes[length];
  }

  CONSTANT_NameAndType_info {
      u1 tag（12 0xc）;
      u2 name_index;
      u2 descriptor_index;
  }

  method_info {
      u2             access_flags;
      u2             name_index;
      u2             descriptor_index;
      u2             attributes_count;
      attribute_info attributes[attributes_count];
  }

  attribute_info {
      u2 attribute_name_index;
      u4 attribute_length;
      u1 info[attribute_length];
  }
```

#### tag

常量类型  | tag 值 |	说明
:----- | :--| :--
CONSTANT_Class	|7（0x7）|类
CONSTANT_Fieldref	|9（0x9）|字段
CONSTANT_Methodref	|10（0xa）|方法	
CONSTANT_InterfaceMethodref	|11（0xb）	|接口方法
CONSTANT_String	|8	（0x8）
CONSTANT_Integer	|3（0x3）	
CONSTANT_Float|	4（0x4）	
CONSTANT_Long|	5（0x5）	
CONSTANT_Double|	6（0x6）	
CONSTANT_NameAndType	|12（0xc）		
CONSTANT_Utf8|	1（0x1）	
CONSTANT_MethodHandle|	15（0xf）		
CONSTANT_MethodType|	16（0x10）	
CONSTANT_Dynamic	|17（0x11）	
CONSTANT_InvokeDynamic|	18（0x12）	
CONSTANT_Module|	19（0x13）	
CONSTANT_Package|	20（0x14）

### 解析后的样子
```cpp
cafe babe // 魔数

0000 // 副版本号
0034 // 主版本号

0016 // 常量池计数器

(1) 0a 0003 0013 // Methodref

(2) 07 0014  // Class
(3) 07 0015  // Class

////// Utf8
(4) 01 0006 3c69 6e69 743e
(5) 01 0003 28 2956
(6) 01 0004 43 6f64 65
(7) 01 000f 4c69 6e65 4e75 6d62 6572 5461 626c 65
(8) 01 0012 4c6f 6361 6c56 6172 6961 626c 6554 6162 6c65
(9) 01 0004 74 6869 73
(10) 01 0006 4c4d 6169 6e3b
(11) 01 000c 64 6f75 626c 654c 6f63 616c 73
(12) 01 0005 2844 4429 44
(13) 01 0002 6431
(14) 01 0001 44
(15) 01 0002 64 32
(16) 01 0010 4d65 7468 6f64 5061 7261 6d65 7465 7273
(17) 01 000a 53 6f75 7263 6546 696c 65
(18) 01 0009 4d61 696e 2e6a 6176 61

(19) 0c 0004 0005  // NameAndType

(20) 01 0004 4d 6169 6e // Utf8
(21) 01 0010 6a61 7661 2f6c 616e 672f 4f62 6a65 6374 // Utf8

0021 // access_flags 类的访问标志

0002 // this_class

0003 // super_class

0000 // interfaces_count

0000 // fields_count

0002 // methods_count  2个方法

///// 方法1
0001 // access_flags
0004 // name_index
0005 // descriptor_index
0001 // attributes_count

0006      // attribute_name_index
0000 002f // attribute_length  0x2f = 47
0001 0001
0000 0005
2ab7 0001
b100 0000
0200 0700
0000 0600
0100 0000
0500 0800
0000 0c00
0100 0000
0500 0900 0a00 00

///// 方法2
0000 // access_flags
000b // name_index
000c // descriptor_index
0002 // attributes_count

0006       // attribute_name_index
00 0000 42 // attribute_length  0x42 = 66

00 0400 0500 0000
0427 2963 af00 0000 0200
0700 0000 0600 0100 0000
0800 0800 0000 2000 0300
0000 0400 0900 0a00 0000
0000 0400 0d00 0e00 0100
0000 0400 0f00 0e00 03

0010
00 0000 09

02 000d 0000 000f 0000

0001 // attributes_count

00 11     // attribute_name_index
0000 0002 // attribute_length

0012
```
#### 简单验证
就拿最后一个属性的字节来验证吧

```c
00 11     // attribute_name_index
0000 0002 // attribute_length

0012
```

attribute_name_index
```c
0012     // 0x0012 = 18    // 01 0009 4d61 696e 2e6a 6176 61
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200804201833306.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)
attribute_length
```c
0000 0002
```

attribute_name_index  
```c
00 11     // 0x0011 = 17  // 01 000a 53 6f75 7263 6546 696c 65
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200804201927530.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200804201222936.png)

👌 完全一致，磕磕碰碰，终于还是给解了 😁


## 参考
[官方文档](https://docs.oracle.com/javase/specs/jvms/se14/html/index.html)





