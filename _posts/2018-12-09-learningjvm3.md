---
layout: post
title:  "类文件结构与虚拟机类加载机制"
subtitle: "JVM笔记3"
header-img: "img/post-bg-universe.jpg"
tag: 
    - JVM笔记
---
**目录**
* content
{:toc}

# 平台无关性
* `平台无关性`是JVM所具有的另一重要特性。这些虚拟机可以载入和执行同一种平台无关的字节码，从而实现了程序的“一次编写，到处运行”。  
* 各种平台的虚拟机与所有平台都同一使用的程序存储格式--字节码是构成平台无关性的基石。
* **Java虚拟机不和包括Java在内的任何语言绑定**，它只与“Class 文件”这种特定的二进制文件格式所关联，Class文件中包含了Java虚拟机指令集和符号表以及若干其他辅助信息。

# Class类文件的结构
* Class文件时一组以8位字节为基础单位的二进制流，各个数据严格按照顺序紧凑地排列在Class文件之中，中间没有添加任何分隔符，这使得整个Class文件中存储地内容几乎全是程序运行地必要数据，没有空隙存在。  
* Class文件格式采用一种类似于C语言结构体地伪结构来存储数据，这种伪结构只有两种数据类型：`无符号数`和`表`。  
**无符号数**：属于基本的数据类型，以u1、u2、u4、u8来分别代表1个字节、2个字节、3个字节、4个字节和8个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值。  
**表**：表是由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性地以“_info”结尾。表用于描述有层次关系地复合结构地数据，整个Class文件本质上就是一张表。  
![](\img\in-post\L-JVM\JVM3-1.PNG)  

## 魔数与Class文件的版本
* 每个Class文件的头4个字节称为`魔数`（Magic Number），它的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。
* 很多文件存储标准中都使用魔数来进行身份识别,Class的魔数值为：`0xCAFEBABE`。
* 第5和第6字节是次版本号（Minor Version），第7和第8字节是主版本号（Major Version）。版本向下兼容。

## 常量池
* 常量池在主次版本号之后，可以理解为Class文件之中的资源仓库。
* 常量池入口需要放置一项u2类型的数据，代表常量池容量计数值（constant_pool_count），从1开始计数。
* 常量池主要存放两大类常量：`字面量`（Literal）和`符号引用`（Symbolic References）。字面量比较接近于Java语言层面的常量概念，如文本字符串、被声明为final的常量等。而符号引用则属于编译原理方面的概念，包括下面三类常量：  
1. 类和接口的全限定名
2. 字段的名称和描述符
3. 方法的名称和描述符
* Java代码进行Javac编译时并不像c与c++连接这一步骤，而是在虚拟机加载Class文件的时候进行`动态连接`。在Class文件中不会保存各个方法和字段的最终内存布局信息，这些符号引用还要经过转换才能直接被虚拟机使用。
* 常量池每一个常量都是一个表，公有11种结构各不相同的表结构数据，它们开始的第一位是一个u1类型的标志位（tag，取值1到12，缺少标志为2的数据类型），代表当前这个常量属于哪种常量类型。如下图  
![](\img\in-post\L-JVM\JVM3-2.PNG)  

## 访问标志
* 常量池结束之后，紧接着的2个字节代表访问标志（acess_flags），**用于识别一些类或接口层次的访问信息**，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是类，是否被声明为fianl等。具体如下图：
![](\img\in-post\L-JVM\JVM3-3.PNG)  

## 类索引、父类索引与接口索引集合
* 类索引（this_class）和父类索引（super_class）都是一个u2类型的数据，而接口索引集合（interfaces）是一组u2类型的数据的集合。
* 类索引用于确定这个类的`全限定名`，父类索引用于确定这个类的父类的`全限定名`。它们各自指向一个类型为CONSTANT_Class_info的类描述常量，通过CONSTANT_Class_info类型的常量种的索引值找到定义在CONSTANT_Utf8_info类型的常量种的全限定字符串。
* 接口索引集合，入口第1项---u2类型的接口计数器（interfaces_count），表示索引表的容量。若无接口则为0。

## 字段表集合
* 用于描述`接口或类中声明的变量`。不包括方法内部的变量。
* 包括的信息有：字段的作用域（public、private、protected修饰符）、是类级变量还是实例级变量（static修饰符）、可变性（final）、并发可见性（volatile修饰符，是否强制从主内存读写）、可否序列化（transient修饰符）、字段数据类型（基本类型、对象、数组）、字段名称。用标志位来表示。
* 字段的名字、字段被定义为什么数据类型，引用常量池中的常量来描述。

## 方法表集合
* Class文件存储格式中堆方法的描述与对字段的描述几乎采用完全一致的方式。
* 因为volatile、transient关键字不能修饰方法，所以方法表的访问标志中没有这些。与之相对synchronized、native、strictfp和abstract可以修饰方法，所以方法表的访问标志中增加了这几个标志。
* 方法里的Java代码，经过编译器编译成字节码指令之后，存放在方法属性表中一个名为`Code`的属性里面。

## 属性表集合
* 在Class文件中、字段表、和方法表中都可以携带自己的属性表集合、以用于描述某些场景专有的信息。
* 在《Java虚拟机规范（第2版）》中预定义了9项虚拟机应当能识别的属性：  
1. **Code属性：**Java程序的方法体里面的代码经过Javac编译器处理之后，最终变为字节码指令存储在Code属性内。接口或抽象类的方法不存在Code属性。Code属性是Class文件中最重要的一个属性，在整个Class文件里，Code属性用于描述`代码`，所有其他数据项目哟关于描述元数据（Metadata、包括类、字段、方法定义及其他信息）。
2. **Exception属性：**在方法表中与Code属性平级的一项属性，用于列举出方法中可能抛出的受查异常，也就是throws关键字后面列举的异常。
3. **LineNumberTable属性：**用于描述Java源码行号与字节码行号（字节码的偏移量）之间的对应关系。
4. **LocalVariableTable属性：**用于描述栈帧中局部变量表中变量与Java源码定义的变量之间的关系，在Javac中可以使用-g：none或-g：vars选项来取消或要求生成这项信息。
5. **SourceFile属性：**用于记录生成这个Class的源码文件名称，在Javac中可以使用-g：none或-g：source选项来关闭或要求生成这项信息。
6. **ConstantValue属性：**用于通知虚拟机自动为静态变量赋值。只有被static关键字修饰的变量（类变量）才可以使用这项属性。
7. **InnerClasses属性：**用于记录内部类与宿主类之间的关联。如果一个类中定义了内部类，那编译器将会为它及它所包含的内部类生成InnerClasses属性。
8. **Deprecated及Synthetic属性：**都属于标志类型的布尔属性，只存在有和没有的区别。Deprecated属性用于表示某个类、字段或方法，已经被程序作者定位不再推荐使用，它可以在代码中使用@deprecated注释进行设置。Synthetic代码此字段或方法并不是由Java源码直接产生的，而是由编译器自行添加的。
9. **属性随着JDK的发展，在不断的添加中**

# 虚拟机类加载机制
虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化、最终形成可以被虚拟机直接使用的Java类型，这就是`虚拟机的类加载机制`。

 `注意`  
* 在实际情况中，每个Class文件都有可能代表Java语言中的一个类或接口。
* Class文件并非指Class必须存在于具体磁盘中的某个文件，这里说的Class文件指的是一串二进制的字节流，无论以何种形式存在都可以。

## 类加载的时机

*** 
**填坑中...**
