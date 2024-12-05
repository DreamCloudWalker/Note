## 多态实现原理

#### 1. 静态多态（重载）

注意，不能通过返回值重载

原理：c++函数名称修饰(不同函数会被修饰成不同的函数名)，在编译和汇编阶段。

编译过程分为： 

1. 预编译： 头文件函数拷贝到源文件
2. 编译 ： 语法分析，同时进行符号汇总
3. 汇编 ： 生成函数名到函数地址的映射，方便之后通过函数名找到函数定义位置，从而执行函数
4. 链接： 将多个文件的符号表汇总合并

#### 2. 动态多台（虚函数重写）

原理：

早绑定: 编译器编译时已经知道对象的内存地址

晚绑定：若类有虚函数，会生成一个虚函数表（一维数组，存放了虚函数地址），类对象构造时会初始化该虚表指针。



## extern “C”

extern 是C/C++ 语言中表明函数和全局变量作用范围的关键字，该关键字告诉编译器，其声明的函数和变量可以在本模块或其它模块中使用。

被extern "C" 修饰的变量和函数是按照C语言方式编译和连接的。

extern “C” 这个声明的真实目的:解决名字匹配问题，实现C++与C的混合编程。



## const 修饰指针的三种效果

const int*p=&a；
当把 const 放最前面的时候，它修饰的就是 *p，那么 *p 就不可变。*p 表示的是指针变量 p 所指向的内存单元里面的内容，此时这个内容不可变。其他的都可变，如 p 中存放的是指向的内存单元的地址，这个地址可变，即 p 的指向可变。但指向谁，谁的内容就不可变。

int*const p=&a；
此时 const 修饰的是 p，所以 p 中存放的内存单元的地址不可变，而内存单元中的内容可变。即 p 的指向不可变，p 所指向的内存单元的内容可变。

const int*const p=&a；
此时 *p 和 p 都被修饰了，那么 p 中存放的内存单元的地址和内存单元中的内容都不可变。

http://c.biancheng.net/view/218.html



## c++ 指针和引用的区别

指针是一个变量，只不过这个变量存储的是一个地址，指向内存的一个存储单元；而引用跟原来的变量实质上是同一个东西，只不过是原变量的一个别名而已。

函数形参是指针的指针传递本质也是值传递，只是传递的值是地址。



## malloc/free 与 new/delete

如果用free释放“new创建的动态对象”，那么该对象因无法执行析构函数而可能导致程序出错。如果用delete释放“malloc申请的动态内存”，理论上讲程序不会出错，但是该程序的可读性很差。所以new/delete，malloc/free必须配对使用。

其中new/delete是C++的运算符，使用时会调用类的构造/析构函数。

malloc分配内存原理：是库函数，128k大小以下分配先从内存池中查看是否有复用对象，如果有从内存池中获取分配并返回。如果没有则通过brk系统调用从堆中分配<font color="red">虚拟内存</font>。在内存被使用之前，不会映射到物理内存。 大于128k的不看内存池，直接通过mmap系统调用从文件映射区获取。

堆上分配的内存，是从低到高。mmap是从高到低。

虚拟内存和物理内存之间有页表。

free如何知道类型或要释放的空间大小？在malloc内存分配时，会在分配块的前面或后面储存<font color="red">元数据</font>，里面包括块大小和使用状态信息。free根据系统访问元数据知道要释放多大的内存。



## C++ 虚函数表

每个包含了虚函数的类都包含一个虚表。

一个类继承了包含虚函数的基类，那么这个类也拥有自己的虚表。

虚表是一个指针数组，其元素是虚函数的指针，每个元素对应一个虚函数的函数指针。

虚表是属于类的，而不是属于某个具体的对象，一个类只需要一个虚表即可。同一个类的所有对象都使用同一个虚表。

为了指定对象的虚表，对象内部包含一个虚表的指针，来指向自己所使用的虚表。



## vector push_back 和 emplace_back 区别

区别主要是针对对象而言，emplace_back 可以减少一次拷贝构造函数的调用。



## vector＜bool＞不是存储bool类型元素的容器

```
//创建一个 vector<bool> 容器
vector<bool>cont{0,1};
//试图将指针 p 指向 cont 容器中第一个元素
bool *p = &cont[0];
```

以上代码无法编译通过。

为了节省空间，vector底层在存储各个 bool 类型值时，每个 bool 值都只使用一个比特位（二进制位）来存储。也就是说在 vector底层，一个字节可以存储 8 个 bool 类型值。

`std::vector<bool>` 是 C++标准库中 `std::vector` 模板类的一个特例化版本，专门用于存储布尔值。这种特殊的实现与其他类型的 `std::vector` 有几个显著的不同之处。

1. **位压缩**:
   - 一般情况下，`std::vector<T>` 会为每个元素分配一个独立的内存块，大小为 `sizeof(T)`。但是，对于 `std::vector<bool>`，它并不是这样处理的。
   - `std::vector<bool>` 使用了位压缩技术，每个布尔值只占用一个位，而不是一个字节。这使得 `std::vector<bool>` 在存储大量布尔值时非常高效。
2. **特殊的接口**:
   - 由于 `std::vector<bool>` 是特殊化的实现，它并不直接提供常规的引用操作。例如，获取 `std::vector<bool>` 中某个元素的引用，实际上会返回一个代理对象（proxy object），而不是布尔类型的引用。这使得它的使用方式与其他类型的 `std::vector` 稍微不同。





## deque 容器的存储结构

vector 容器采用连续的线性空间不同，deque 容器存储数据的空间是由一段一段等长的连续空间构成，各段空间之间并不一定是连续的，可以位于在内存的不同区域。

当 deque 容器需要在头部或尾部增加存储空间时，它会申请一段新的连续空间，同时在 map 数组的开头或结尾添加指向该空间的指针，由此该空间就串接到了 deque 容器的头部或尾部。



## STL map 和 unordered_map 的区别

map 是一种有序的容器，底层是用红黑树实现的，它可以做到 O(logn) 时间完成查找、插入、删除元素的操作。

unordered_map是一种无序的容器，底层是用哈希表实现的(哈希表-维基百科)，哈希表最大的优点是把数据的查找和存储时间都大大降低。

## STL unordered_map 原理

C++ STL 标准库中，不仅是 unordered_map 容器，所有无序容器的底层实现都采用的是哈希表存储结构。存储结构：数组+链表，数组用来保存链表的头指针，各个键值对存储在链表的结点中

当有新键值对存储到无序容器中时，整个存储过程分为如下几步：
将该键值对中键的值带入设计好的哈希函数，会得到一个哈希值（一个整数，用 H 表示）；
将 H 和无序容器拥有桶的数量 n 做整除运算（即 H % n），该结果即表示应将此键值对存储到的桶的编号；
建立一个新节点存储此键值对，同时将该节点链接到相应编号的桶上。



## C++ RVO Return Value Optimization

```cpp
#include <stdio.h>

struct Message
{
    Message()
    { 
        printf("Message::Message() is called\n"); 
    }
    Message(const Message &)
    {
        printf("Message::Message(const Message &msg) is called\n");
    }
    Message& operator=(const Message &)
    {
        printf("Message::operator=(const Message &) is called\n");
    }
    ~Message()
    {
        printf("Message::~Message() is called\n");
    }
    int a;
    int b;
    int c;
    int d;
    int e;
    int f;
};

Message getMessage()
{
    Message result;
    result.a = 0x11111111;

    return result;
}

int main()
{
    Message msg = getMessage();
    return 0;
}
```

你认为运行时会输出什么呢？是不是这样：

```
Message::Message() is called
Message::Message(const Message &msg) is called
Message::~Message() is called
Message::~Message() is called
```

RVO 优化之后：

```
Message::Message() is called
Message::~Message() is called
```

它存在的目的是优化掉不必要的拷贝构造函数的调用，基本手段是直接将返回的对象构造在调用者栈帧上。



## Android HIDL

AIDL常用于连接App和Framework，HIDL则是用来连接Framework和HAL，AIDL使用Binder通信，HIDL则使用HwBinder通信，他们都是通过Binder驱动完成通信，只不过两个Binder域不一样。

将 Framework 与 HAL 独立出来，简化Android系统升级的影响与难度。

app -> java framework -> jni -> hidl service -> HAL



## Android Malloc Debug 原理

Malloc Debug 可用于检查内存泄漏、内存越界、double free、use after free

1. 代理内存分配和释放；
2. 额外申请一部分内存用于保存内存的信息（Header）和设置内存边界（guard）；
3. 调用栈跟踪。



## Android ASan 原理

主要是通过 shadow 内存，用于描述应用程序内存区域是否可被读写。

程序申请的内存的前后，各增加一个redzone区域（n * 8bytes），用户申请的内存对应的shadow内存会被标记成可读写的，而redzone区域内存对应的shadow内存则会被标记成不可读写的。

free对象时，asan不会立即把这个对象的内存释放掉，而是写入1个负数到该对象的shadown内存中，即将该对象成不可读写的状态。

并将它记录放到一个隔离区(book keeping)中, 这样当有野指针或use-after-free的情况时，就能跟进shadow内存的状态，发现程序的异常；一段时间后如果程序没有异常，就会再释放隔离区中的对象。

编译器在对每个变量的load/store操作指令前都插入检查代码，确认是否有overflow、underflow、use-after-free等问题。



## C++ 仿函数

重载函数调用操作符的类，其对象常称为函数对象(function object)，也叫仿函数(functor)，使得类对象可以像函数那样调用。

函数对象通常不定义构造和析构函数，所以在构造和析构时不会发生任何问题，避免了函数调用时的运行时问题。

lambda 表达式的内部实现其实也是仿函数。

C++11中的lambda表达式在使用时确实会被转换成仿函数。具体来说，当你定义一个lambda表达式时，编译器会生成一个具有重载 `operator()` 的匿名类，从而使得这个lambda表达式成为一个可调用对象。

以下是lambda表达式的基本语法：

```c++
[capture](parameters) -> return_type { body }
```

1. **捕获列表 (`capture`)**: 指定了lambda可以访问外部变量的方式。
2. **参数列表 (`parameters`)**: 指定了lambda接受的参数。
3. **返回类型 (`return_type`)**: （可选）指定返回值的类型。
4. **函数体 (`body`)**: lambda的实现代码。

#### 示例与分析

```c++
#include <iostream>

int main() {
    int x = 10;
    
    // 定义一个lambda表达式
    auto add = [x](int y) { return x + y; };
    
    // 调用lambda
    std::cout << add(5) << std::endl; // 输出 15
    return 0;
}
```

在这个例子中：

- `add` 是一个lambda表达式，它被定义为一个可以调用的对象。它捕获了外部变量 `x`，并接受一个参数 `y`，返回 `x + y` 的结果。

#### 编译器的转换

在编译过程中，编译器会将这个lambda表达式转换为一个实现了 `operator()` 的匿名类。例如，编译器可能会生成类似以下的结构：

```c++
class LambdaClass {
public:
    LambdaClass(int x) : x_(x) {}

    int operator()(int y) const {
        return x_ + y;
    }

private:
    int x_;
};

// 使用
LambdaClass add(x);
std::cout << add(5) << std::endl; // 输出 15
```

## C++ lambda 表达式

实际上是一个函数对象（仿函数、匿名类对象函数），内部重载了函数调用操作符；
lambda 表达式就是一个函数（匿名函数），也就是一个没有函数名的函数。为什么不需要函数名呢，因为我们直接（一次性的）用它，嵌入式用的它，不需要其他地方调用它
lambda 表达式也叫闭包。闭就是封闭的意思（封闭就是其他地方都不调用它），包就是函数。

lambda 表达式其实就是一个函数对象，他内部创建了一个重载()操作符的类。

lambda 表达式的简单语法如下：capture -> return value { body }, 只有 [capture] 捕获列表和 { body } 函数体是必选的，其他可选。

## std::function,std::bind和lambda关系

std::function 是抽象了函数参数和返回值的类模板。把任意函数包装成一个对象，动态绑定。保存普通函数、类的静态成员函数或仿函数。





## C++ 20 协程

协程就是一段可以挂起（suspend）和恢复（resume）的程序，一般而言，就是一个支持挂起和恢复的函数。

不阻塞当前执行的线程，把当前函数（协程）执行的位置存起来，在将来某个时间点又读取出来继续执行的。



## C++ 智能指针

C++中智能指针的实现主要依赖于两个技术概念：

1、析构函数，对象被销毁时会被调用的一个函数，对于基于栈的对象而言，如果对象离开其作用域则对象会被自动销毁，而此时析构函数也自动会被调用。

2、引用计数技术，维护一个计数器用于追踪资源(如内存)的被引用数，当资源被引用时，计数器值加1，当资源被解引用时，计算器值减1。

3、操作符重载。

智能指针的大致实现原理就是在析构函数中，检查所引用对象的引用计数，如果引用计数为0，则真正释放该对象内存。

```cpp
// asdfa.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"
#include<iostream>
using namespace std;
//引用计数类
class counter
{
public:
    counter(){}
    counter(int parCount) :count(parCount){}
    void increaseCount() { count++; }
    void decreasCount(){ count--; }
    int  getCount(){ return count; }
private:
    int count;	// TODO count需要是atomic保证线程安全
};

//智能指针
template<class T>
class SmartPointer
{
public:
    explicit  SmartPointer(T* pT) :mPtr(pT), pCounter(new counter(1)){}
    explicit  SmartPointer():mPtr(NULL),pCounter(NULL){}
    ~SmartPointer()     //析构函数，在引用计数为0时，释放原指针内存
    {
        if (pCounter != NULL)
        {
            pCounter->decreasCount();
            if (pCounter->getCount() == 0)
            {
                delete pCounter;
                delete mPtr;
                pCounter = NULL;    //将pCounter赋值为NULL,防止悬垂指针
                mPtr = NULL;
                cout << "delete original pointer" << endl;
            }
        }
    }

    SmartPointer(SmartPointer<T> &rh)   //拷贝构造函数，引用加1
    {
        this->mPtr=rh.mPtr;
        this->pCounter = rh.pCounter;
        this->pCounter->increaseCount();
    }

    SmartPointer<T>& operator=(SmartPointer<T> &rh) //赋值操作符，引用加1
    {
        if (this->mPtr == rh.mPtr)
            return *this;
        this->mPtr = rh.mPtr;
        this->pCounter = rh.pCounter;
        this->pCounter->increaseCount();
        return *this;
    }
    T& operator*()          //重载*操作符
    {
        return *mPtr;
    }

    T* operator->()         //重载->操作符
    {
        return p;
    }
    T* get()
    {
        return mPtr;
    }
private:
    T* mPtr;
    counter* pCounter;
};
int _tmain(int argc, _TCHAR* argv[])
{
    SmartPointer<int> sp1(new int(10));
    SmartPointer<int> sp2 = sp1;
    SmartPointer<int> sp3;
    sp3 = sp2;
    return 0;
}
```



## C++ 右值引用

右值引用是C++11中引入的新特性 , 它实现了转移语义和精确传递。

它的主要目的有两个方面：

消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率。
能够更简洁明确地定义泛型函数。

右值引用就是必须绑定到右值的引用，他有着与左值引用完全相反的绑定特性，我们通过 && 来获得右值引用。

右值引用的基本语法type &&引用名 = 右值表达式；

右值有一个重要的性质——只能绑定到一个将要销毁的对象上。举个例子：

```
int  &&rr = i;  //错误，i是一个变量，变量都是左值
int &&rr1 = i *42;  //正确，i*42是一个右值
```

右值引用和左值引用的区别:
左值可以寻址，而右值不可以。
左值可以被赋值，右值不可以被赋值，可以用来给左值赋值。
左值可变,右值不可变（仅对基础类型适用，用户自定义类型右值引用可以通过成员函数改变）。

## C++ 完美转发

所谓的完美转发，是指std::forward会将输入的参数原封不动地传递到下一个函数中，这个“原封不动”指的是，如果输入的参数是左值，那么传递给下一个函数的参数的也是左值；如果输入的参数是右值，那么传递给下一个函数的参数的也是右值。

防止在参数传递的过程中，右值引用会变成左值引用，从而调用拷贝构造而不是移动构造。

无论左值引用类型的变量还是右值引用类型的变量，都是左值，因为它们有名字。



## C++ 11 constexpr

constexpr是C++11中新增的关键字，其语义是“常量表达式”，也就是<font color="red">在编译期可求值的表达式</font>。最基础的常量表达式就是字面值或全局变量/函数的地址或sizeof等关键字返回的结果，而其它常量表达式都是由基础表达式通过各种确定的运算得到的。constexpr值可用于enum、switch、数组长度等场合。

constexpr所修饰的变量一定是编译期可求值的，所修饰的函数在其所有参数都是constexpr时，一定会返回constexpr。

```
constexpr int Inc(int i) {
    return i + 1;
}

constexpr int a = Inc(1); // ok
constexpr int b = Inc(cin.get()); // !error
constexpr int c = a * 2 + 1; // ok
```

constexpr还能用于修饰类的构造函数，即保证如果提供给该构造函数的参数都是constexpr，那么产生的对象中的所有成员都会是constexpr，该对象也就是constexpr对象了，可用于各种只能使用constexpr的场合。注意，constexpr构造函数必须有一个空的函数体，即所有成员变量的初始化都放到初始化列表中。

```
struct A {
    constexpr A(int xx, int yy): x(xx), y(yy) {}
    int x, y;
};

constexpr A a(1, 2);
enum {SIZE_X = a.x, SIZE_Y = a.y};
```

constexpr的好处：

是一种很强的约束，更好地保证程序的正确语义不被破坏。
编译器可以在编译期对constexpr的代码进行非常大的优化，比如将用到的constexpr表达式都直接替换成最终结果等。
相比宏来说，没有额外的开销，但更安全可靠



## C++ .hpp

定义与实现都包含在同一文件，调用者只需要include该hpp文件即可。

非常适合用来编写公用的开源库。

不可包含全局对象和全局函数。

类之间不可循环调用。

不可使用静态成员。



## C++ 锁

### std::recursive_mutex 嵌套锁/递归锁/重入锁

std::recursive_mutex 允许同一个线程对互斥量多次上锁（即递归上锁），来获得对互斥量对象的多层所有权，std::recursive_mutex 释放互斥量时需要调用与该锁层次深度相同次数的 unlock()，可理解为 lock() 次数和 unlock() 次数相同，除此之外，std::recursive_mutex 的特性和 std::mutex 大致相同。

### std::timed_mutex、 recursive_timed_mutex

timed_mutex.try_lock_for(std::chrono::milliseconds(200)); //在指定的时限中获取到锁则返回 true , 否则返回 false

### std::shared_timed_mutex 与 std::shared_lock（C++14）

C++14 通过 std::shared_timed_mutex 和 std::shared_lock 来实现读写锁，保证多个线程可以同时读，但是写线程必须独立运行，写操作不可以同时和读操作一起进行。

```cpp
struct ThreadSafe {
    mutable std::shared_timed_mutex mutex_;
    int value_;

    ThreadSafe() {
        value_ = 0;
    }

    int get() const {
        std::shared_lock<std::shared_timed_mutex> loc(mutex_);
        return value_;
    }

    void increase() {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);
        value_ += 1;
    }
};
```



## C++ 20 Concepts

用于约束模板函数和模板类的模板参数。
https://zhuanlan.zhihu.com/p/107610017



## C++ 20 Ranges

ranges 可以省掉很多循环，包括多重循环，写出来的代码又简单，可读性又好。

```cpp
vector<int> data{4, 3, 4, 1, 8, 0, 8}; 
vector<int> result = data | actions::sort | actions::unique;
```





