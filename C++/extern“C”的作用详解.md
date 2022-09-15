## 简介

C++ 语言的创建初衷是 "a better C"，但是这并不意味着 C++ 中类似 C 语言的全局变量和函数所采用的编译和连接方式与 C 语言完全相同。

作为一种欲与 C 兼容的语言， C++ 保留了一部分过程式语言的特点（被世人称为"不彻底地面向对象"），因而它可以定义不属于任何类的全局变量和函数。

但是， **C++ 毕竟是一种面向对象的程序设计语言，为了支持函数的重载， C++ 对全局函数的处理方式与 C 有明显的不同。本文将介绍 C++ 中如何通过 extern "C" 关键字支持 C 语言。**



# 前言

如果你做过Android frameworks开发的话，或者做过jni开发的话，肯定经常看到如下代码：

```
#ifdef __cplusplus
extern "C" {
#endif
// 正式定义。。。
#ifdef __cplusplus
}
#endif
```

如果你是个C语言开发者，看到这个时可能感到疑惑，为什么要有一个extern “C”呢？



# 作用

为什么C语言开发者不常见到这种语法呢，因为这种语法其实是C++中的。

extern "C"的主要作用就是为了实现C++代码调用其他C语言代码。加上extern "C"后，会**指示编译器这部分代码按C语言的进行编译**，而不是C++的。由于C++支持函数重载，因此编译器编译函数的过程中会将函数的参数类型添加到编译后的代码中，而不仅仅是函数名；而C语言并不支持函数重载，因此编译C语言代码的函数时不会带上函数的参数类型，一般只包括函数名。

**extern “C”有两个含义：**

1. 被extern "C"限定的函数或变量是extern类型。
2. 被extern "C"修饰的变量和函数是按照C语言方式进行编译。



## 问题的引出

某企业曾经给出如下的一道面试题

为什么标准头文件都有类似以下的结构？

```
//incvxworks.h
#ifndef __INCvxWorksh
#define __INCvxWorksh

#ifdef __cplusplus
extern "C" {
#endif

/*...*/

#ifdef __cplusplus
}
#endif

#endif /* __INCvxWorksh */
```

问题分析 对于上面问题，显然，头文件中的编译宏 #ifndef __INCvxWorksh 、 #define __INCvxWorksh 、 #endif 的作用是防止该头文件被重复引用。

那么，

```
#ifdef __cplusplus
extern "C" {
#endif
和
#ifdef __cplusplus
}
#endif

```

的作用又是什么呢？我们将在后面对此进行详细说明。



## 关于 extern "C"

前面的题目中的 __cplusplus 宏，是用来识别编译器的，也就是说，将当前代码编译的时候，是否将代码作为 C++ 进行编译。如果是，则定义了 __cplusplus 宏。更多内容，这里就不详细说明了。 

而题目中的 extern "C" 包含双重含义，从字面上即可得到：首先，被它修饰的目标是 extern 的；其次，被它修饰的目标是 C 的。

具体如下：被 extern "C" 限定的函数或变量是 extern 类型的。

extern 是 C/C++ 语言中表明函数和全局变量作用范围（可见性）的关键字，该关键字告诉编译器，其声明的函数和变量可以在本模块或其它模块中使用。

注意，语句 extern int a; 仅仅是对变量的声明，其并不是在定义变量 a ，声明变量并未为 a 分配内存空间。定义语句形式为 int a; ，变量 a

在所有模块中作为一种全局变量只能被定义一次，否则会出现连接错误。

在引用全局变量和函数之前，必须要有这个变量或者函数的声明（或者定义）。通常，在模块的头文件中对本模块提供给其它模块引用的函数和全局变量以关键字 extern 声明。

例如，如果模块 B 欲引用该模块 A 中定义的全局变量和函数时只需包含模块 A 的头文件即可。这样，模块B中调用模块 A 中的函数时，在编译阶段，模块 B 虽然找不到该函数，但是并不会报错；它会在连接阶段中从模块 A 编译生成的目标代码中找到此函数。

与 extern 对应的关键字是 static ，被它修饰的全局变量和函数只能在本模块中使用。因此，一个函数或变量只可能被本模块使用时，其不可能被 extern "C" 修饰。被 extern "C" 修饰的变量和函数是按照 C 语言方式编译和连接的。

首先看看 C++ 中，在未加 extern "C" 声明时，对类似 C 的函数是怎样编译的。

作为一种面向对象的语言， C++ 支持函数重载，而过程式语言 C 则不支持。所以，函数被 C++ 编译后在符号库中的名字与 C 语言的有所不同。例如，假设某个函数的原型为：void foo( int x, int y );

该函数被 C 编译器编译后在符号库中的名字为 _foo ，而 C++ 编译器则会产生像 _foo_int_int 之类的名字（不同的编译器可能生成的名字不同，但是都采用了相同的机制，生成的新名字称为 mangled name ）。

_foo_int_int 这样的名字包含了函数名、函数参数数量及类型信息， C++ 就是靠这种机制来实现函数重载的。例如，在 C++ 中，函数 void foo( int x, int y ) 与 void foo( int x, float y ) 编译生成的符号是不相同的，后者为 _foo_int_float 。

同样地， C++ 中的变量除支持局部变量外，还支持类成员变量和全局变量。用户所编写程序的类成员变量可能与全局变量同名，我们以 . 来区分。

而本质上，编译器在进行编译时，与函数的处理相似，也为类中的变量取了一个独一无二的名字，这个名字与用户程序中同名的全局变量名字不同。

其次，看看在未加 extern "C" 声明时，是如何连接的。假设在 C++ 中，模块 A 的头文件如下：

```c++
//模块A头文件&emsp;moduleA.h
#ifndef MODULE_A_H
#define MODULE_A_H
int foo( int x, int y );
#endif

在模块 B 中引用该函数：
// 模块B实现文件&emsp;moduleB.cpp
#include "moduleA.h"
foo(2,3);

```

实际上，在连接阶段，连接器会从模块 A 生成的目标文件 moduleA.obj 中寻找 _foo_int_int 这样的符号！对于上面例子，如果 B 模块是 C 程序，而A模块是 C++ 库头文件的话，会导致链接错误；

同理，如果B模块是 C++ 程序，而A模块是C库的头文件也会导致错误。再者，看看加 extern "C" 声明后的编译和连接方式

加 extern "C" 声明后，模块 A 的头文件变为：



// 模块A头文件 moduleA.h 

\#ifndef MODULE_A_H 

\#define MODULE_A_H 

extern "C" int foo( int x, int y ); 

\#endif



在模块 B 的实现文件中仍然调用 foo( 2,3 ) ，其结果，将会是 C 语言的编译连接方式：模块 A 编译生成 foo 的目标代码时，没有对其名字进行特殊处理，采用了 C 语言的方式；连接器在为模块 B 的目标代码寻找 foo(2,3) 调用时，寻找的是未经修改的符号名 _foo 。

如果在模块 A 中函数声明了 foo 为 extern "C" 类型，而模块 B 中包含的是 extern int foo( int x, int y ) ，则模块 B 找不到模块 A 中的函数(因为这样的声明没有使用 extern "C" 指明采用C语言的编译链接方式)；反之亦然。

所以，综上可知， extern "C" 这个声明的真实目的，就是实现 C++ 与 C 及其它语言的混合编程。用法举例

C++ 引用 C 函数的具体例子 在 C++ 中引用 C 语言中的函数和变量，在包含 C 语言头文件（假设为 cExample.h ）时，需进行下列处理：

```c
extern "C"
{
    #include "cExample.h"
}

```

因为， C 库的编译当然是用 C 的方式生成的，其库中的函数标号一般也是类似前面所说的 _foo 之类的形式，没有任何参数信息，所以当然在 C++ 中，要指定使用 extern "C" ，进行 C 方式的声明（如果不指定，那么 C++ 中的默认声明方式当然是 C++ 方式的，也就是编译器会产生 _foo_int_int 之类包含参数信息的、 C++ 形式的函数标号，这样的函数标号在已经编译好了的、可以直接引用的 C 库中当然没有）。

通过头文件对函数进行声明，再包含头文件，就能引用到头文件中声明的函数(因为函数的实现在库中呢，所以只声明，然后链接就能用了)。

而在 C 语言中，对其外部函数只能指定为 extern 类型，因为 C 语言中不支持 extern "C" 声明，在 .c 文件中包含了 extern "C" 时，当然会出现编译语法错误。

下面是一个具体代码：

```c++
/* c语言头文件：cExample.h */
#ifndef C_EXAMPLE_H
#define C_EXAMPLE_H
extern int add(int x,int y);
#endif

/* c语言实现文件：cExample.c */
#include "cExample.h"
int add( int x, int y )
{
    return x + y;
}

// c++实现文件，调用add：cppFile.cpp
extern "C"
{
    #include "cExample.h"
}
int main(int argc, char* argv[])
{
    add(2,3);
    return 0;
}
```

，如果 C++ 调用一个 C 语言编写的 .DLL 时，在包含 .DLL 的头文件或声明接口函数时，应加 extern "C" { } 。

**这个时候，其实 extern "C" 是在告诉 C++ ，链接 C 库的时候，采用 C 的方式进行链接**（即寻找类似 _foo 的没有参数信息的标号，而不是默认的 _foo_int_int 这样包含了参数信息的 C++ 标号了）。

C 引用 C++ 函数的具体例子在C中引用 C++ 语言中的函数和变量时， C++ 的头文件需添加 extern "C" ，但是在 C 语言中不能直接引用声明了 extern "C" 的该头文件(因为C语言不支持 extern "C" 关键字，所以会报编译错误)，应该仅在 C 文件中用 extern 声明 C++ 中定义的 extern "C" 函数(就是 C++ 中用 extern "C" 声明的函数。

在 C 中用 extern 来声明一下，这样 C 就能引用 C++ 的函数了，但是 C 中是不用用 extern "C" 的)。

下面是一个具体代码：

```c++
//C++头文件 cppExample.h
#ifndef CPP_EXAMPLE_H
#define CPP_EXAMPLE_H
extern "C" int add( int x, int y );
#endif

//C++实现文件 cppExample.cpp
#include "cppExample.h"
int add( int x, int y )
{
    return x + y;
}

/* C实现文件 cFile.c
/* 这样会编译出错：#include "cExample.h" */
extern int add( int x, int y );

int main( int argc, char* argv[] )
{
    add( 2, 3 );   
    return 0;
}

```

上面例子， C 实现文件 cFile.c 不能直接用 #include "cExample.h"= 因为 =C 语言不支持 extern "C" 关键字。

这个时候，而在 cppExample.h 中使用 extern "C" 修饰的目的是为了让 C++ 编译时候能够生成 C 形式的符号(类似 _foo 不含参数的形式)，然后将其添加到对应的 C++ 实现库中，以便被 C 程序链接到。

对 __BEGIN_DECLS  和  __END_DECLS  的理解在 C 语言代码中头文件中，经常看到充斥着下面的代码片段：

```c++
1. __BEGIN_DECLS
2. .....
3. .....
4. __END_DECLS

```

其实，这些宏一般都是在标准库头文件中定义好了的，例如我当前机器的 sys/cdefs.h 中大致定义如下：

```c++
1. #if defined(__cplusplus)
2.        #define __BEGIN_DECLS extern "C" {
3.        #define __END_DECLS }
4.        #else
5.        #define __BEGIN_DECLS
6.        #define __END_DECLS
7. #endif

```

这目的当然是扩充 C 语言在编译的时候，按照 C++ 编译器进行统一处理，使得 C++ 代码能够调用 C 编译生成的中间代码。 

由于 C 语言的头文件可能被不同类型的编译器读取，因此写 C 语言的头文件必须慎重。总结extern "C" 只是 C++ 的关键字，不是 C 的所以，如果在 C 程序中引入了 extern "C" 会导致编译错误。 

**被 extern "C" 修饰的目标一般是对一个C或者 C++ 函数的声明从源码上看 extern "C" 一般对头文件中函数声明进行修饰。**

无论 C 程序中的还是 cpp 中的头文件，其函数声明的形式都是一样的（因为两者语法基本一样），对应声明的实现却可能由于相应的程序特性而不同了( C 库和 C++ 库里面当然会不同)。

**extern "C" 这个关键字声明的真实目的，就是实现 C++ 与C及其它语言的混合编程**

**一旦被 extern "C" 修饰之后，它便以 C 的方式工作，可以实现在 C 中引用 C++ 库的函数，也可以 C++ 中引用 C 库的函数。**


*作者：linux服务器开发架构师
链接：https://juejin.cn/post/6844903879570620430*

