# 前言

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/29c0338c4d3847508716a00c9bd5faea~noop.image?_iz=58558&from=article.pc_detail&x-expires=1670479652&x-signature=%2BNHPd6RApy1Jmh4DPpzDOFzT5CI%3D)

《Effective C++ （第三版本）》书中的「条款」这一词，我更喜欢以「细节」替换，毕竟年轻的我们在打 LOL 或 王者的时，总会说注意细节！细节！细节~ —— 细节也算伴随我们的青春的字眼



# 细节 01：尽量以const，enum，inline 替换 #define

> \#define 定义的常量有什么不妥？

首先我们要清楚程序的编译重要的三个阶段：**预处理阶段，编译阶段和链接阶段**。

\#define 是不被视为语言的一部分，它在程序编译阶段中的**预处理阶段**的作用，就是做简单的替换。

如下面的 PI 宏定义，在程序编译时，编译器在预处理阶段时，会先将源码中所有 PI 宏定义替换成 3.14：

```c++
#define PI 3.14
```

程序编译在预处理阶段后，才进行真正的编译阶段。在有的编译器，运用了此 PI 常量，**如果遇到了编译错误，那么这个错误信息也许会提到 3.14 而不是 PI**，这就会让人困惑哪里来的3.14，特别是在项目大的情况下。

> 解决之道：以 const 定义一个常量替换上述的宏（#define）

作为一个语言变量，下面的 const 定义的常量 Pi 肯定会被编译器看到，出错的时候可以很清楚知道，是这个变量导致的问题：

```c++
const double Pi = 3.14;
```

如果是定义常量字符串，则必须要 const 两次，目的是为了防止指针所指内容和指针自身不能被改变：

```c++
const char* const myName = "coding";
```

如果是定义常量 string，则只需要在最前面加一次 const，形式如下：

```c++
const std::string myName("coding");
```

------

> \#define 不重视作用域，所以对于 class 的专属常量，应避免使用宏定义。



还有另外一点宏无法涉及的，就是我们无法利用 #define 创建一个 class 专属常量，因为 #define 并不重视作用域。

对于类里要定义专属常量时，我们依然使用 static + const，形式如下：

```c++
class Student {
private:
    static const int num = 10;
    int scores[num];
};

const int Student::num; // static 成员变量，需要进行声明
```

> 如果不想外部获取到 class 专属常量的内存地址，可以使用 enum 的方式定义常量

enum 会帮你约束这个条件，因为取一个 enum 的地址是不合法的，形式如下：

```c++
class Student {
private:
    enum { num = 10 };
    int scores[num];
};
```

------

> \#define 实现的函数容易出错，并且长相丑陋不易阅读。

另外一个常见的 #define 误用情况是以它实现宏函数，它不会招致函数调用带来的开销，但是用 #define 编写宏函数容易出错，如下用宏定义写的求最大值的函数：

```c++
#define MAX(a, b) ( { (a) > (b) ? (a) : (b); } ) // 求最大值
```

这般长相的宏有着太的缺点，比如在下面调用例子：

```c++
int a = 6, b = 5;
int max = MAX(a++, b);

std::cout << max << std::endl;
std::cout << a << std::endl;
```

输出结果（以下结果是错误的）：

```
7 // 正确的答案是 max 输出 6
8 // 正确的答案是  a  输出 7
```

要解释出错的原因很简单，我们把 MAX 宏做简单替换：

```
int max = ( { (a++) > (b) ? (a++) : (b); } ); // a 被累加了2次！
```

在上述替换后，可以发现 a 被累加了 **2 次**。我们可以通过改进 MAX 宏，来解决这个问题：

```
#define MAX(a, b) ({ \
    __typeof(a) __a = (a), __b = (b); \
    __a > __b ? __a : __b; \
})
```

简单说明下，上述的 __typeof 可以根据变量的类型来定义一个相同类型的变量，如 a 变量是 int 类型，那么 __a 变量的类型也是 int 类型。改进后的 MAX 宏，**输出的是正确的结果，max 输出 6，a 输出 7。**

虽然改进的后 MAX 宏，解决了问题，但是这种宏的长相就让人困惑。

> 解决的方式：用 inline 替换 #define 定义的函数

用 inline 修饰的函数，**也是可以解决函数调用的带来的开销，同时阅读性较高**，不会让人困惑。

下面用用 template inline 的方式，实现上述宏定义的函数：：

```
template<typename T>
inline T max(const T& a, const T& b)
{
    return a > b? a : b;
}
```

max 是一个真正的函数，它遵循**作用域和访问规则**，所以不会出现变量被多次累加的现象。

模板的基础知识内存，可移步到我的旧文进行学习 --> 泛型编程的第一步，掌握模板的特性！

------

> **细节 01 小结 - 请记住**对于单纯常量，最好以 const 对象或 enum 替换 #define；对于形式函数的宏，最好改用 inline 函数替换 #define。



# 细节 02：尽可能使用 const

> const 的一件奇妙的事情是：它允许你告诉编译器和其他程序员**某值应该保持不变**。

**1.** 面对*指针*，你可以指定指针自身、指针所指物，或两者都（或都不）是 const：

```c++
char myName[] = "coding";
char *p = myName;             // non-const pointer, non-const data
const char* p = myName;       // non-const pointer, const data
char* const p = myName;       // const pointer, non-const data
const char* const p = myName; // const pointer, const data
```

- 如果关键词const出现在星号（*）**左**边，表示**指针所指物是常量**（不能改变 *p 的值）；
- 如果关键词const出现在星号（*）**右**边，表示**指针自身是常量**（不能改变 p 的值）；
- 如果关键词const出现在星号（*）**两**边，表示**指针所指物和指针自身都是常量**；

**2.** 面对*迭代器*，你也指定迭代器自身或自迭代器所指物不可被改变：

```c++
std::vector<int> vec;

const std::vector<int>::iterator iter = vec.begin(); // iter 的作用像 T* const
*iter = 10; // 没问题，可以改变 iter 所指物   
++iter;     // 错误! 因为 iter 是 const     

std::vector<int>::const_iterator cIter = vec.begin(); // cIter 的作用像 const T*
*cIter = 10; // 错误! 因为 *cIter 是 const           
++cIter;     // 没问题，可以改变 cIter                        
```

- 如果你希望迭代器自身不可被改动，像指针声明为 const 即可（即声明一个 T* const 指针）； —— **这个不常用**
- 如果你希望迭代器所指的物不可被改动，你需要的是 const_iterator（即声明一个 const T* 指针）。—— **这个常用**

------

> const 最具有威力的用法是面对函数声明时的应用。在一个函数声明式内，const 可以和函数返回值、各参数、成员函数自身产生关联。

**1.** 令*函数返回*一个常量值，往往可以降低因程序员错误而造成的意外。举个例子：

```c++
class Rational { ... };
const Rational operator* (const Rational& lhs, const Rational& rhs);
```

为什么要返回一个 const 对象呢？原因是如果不这样，程序员就能实现这一的暴力行为：

```c++
Rational a, b, c;
if (a * b = c) ... // 做比较时，少了个等号
```

**如果 operator* 返回的 const 对象，可以预防这个没意义的赋值动作**。

**2.** 将 const 实施于*成员函数*的目的，是为了确认该成员函数可作用于 const 对象。理由如下两个：

*理由 1 ：*

它们使得 class 接口比较容易理解，因为可以得知哪个函数可以改动对象而哪些函数不行，见如下例子：

```c++
 class MyString
 {
 public:
     const char& operator[](std::size_t position) const // operator[] for const 对象
     { return text[position]; }
 
     char& operator[](std::size_t position)  // operator[] for non-const 对象
     { return text[position]; }
 private:
    std::string text;
};
```

MyString 的 operator[] 可以被这么使用：

```c++
MyString ms("coding"); // non-const 对象
std::cout << ms[0];   // 调用 non-const MyString::operator[]
ms[0] = 'x';          // 没问题，写一个 non-const  MyString

const MyString cms("coding"); // const 对象
std::cout << cms[0];   // 调用 const MyString::operator[]
cms[0] = 'x';          // 错误！ 写一个 const  MyString
```

注意，上述第 7 行会出错，原因是 cms 是 const 对象，调用的是函数返回值为 const 类型的 operator[] ，我们是不可以对 const 类型的变量或变量进行修改的。

*理由 2 ：*

它们使操作 const 对象成为可能，这对编写高效代码是个关键，因为**改善 C++ 程序效率的一个根本的方法是以 pass by referenc-to-const（const T& a） 方式传递对象**，见如下例子：

```c++
 class MyString
 {
 public:
 
     MyString(const char* str) : text(str)
     { 
         std::cout << "构造函数" << std::endl; 
     }
 
    MyString(const MyString& myString) 
    {
        std::cout << "复制构造函数" << std::endl;
        (*this).text = myString.text;
    }

    ~MyString() 
    { 
        std::cout << "析构函数" << std::endl; 
    }

    bool operator==(MyString rhs) const      // pass by value 按值传递
    {
        std::cout << "operator==(MyString rhs) pass by value" << std::endl;
        return (*this).text == rhs.text;
    }
private:
    std::string text;
};
```

operator== 函数是 pass by value， 也就是按值传递，我们使用它，看下会输出什么：

```c++
 int main()
 {
     std::cout << "main()" << std::endl;
     MyString ms1("coding");
     MyString ms2("coding");
 
     std::cout << ( ms1 == ms2) << std::endl; ;
     std::cout << "end!" << std::endl;
     return 0;
}
```

输出结果：

```c++
 main()
 构造函数
 构造函数
 复制构造函数
 operator==(MyString rhs)  pass by value
 
 析构函数
 end!
 析构函数
 析构函数
```

可以发现在进入 operator== 函数时，发生了「复制构造函」，当离开该函数作用域后发生了「析构函数」。说明「按值传递」，在进入函数时，会产生一个副本，离开作用域后就会消耗，说明这里是存在开销的。

我们把 operator== 函数改成 pass by referenc-to-const 后，可以减少上面的副本开销：

```c++
bool operator==(const MyString& rhs)
{
    std::cout << "operator==(const MyString& rhs)  
        pass by referenc-to-const" << std::endl;
    return (*this).text == rhs.text;
}
```

再次输出的结果：

```c++
main()
构造函数
构造函数
operator==(const MyString& rhs)  pass by referenc-to-const
1
end!
析构函数
析构函数
```

**没有发生复制构造函数，说明 pass by referenc-to-const 比 pass by value 性能高。**

------

> 在 const 和 non-const 成员函数中避免代码重复

假设 MyString 内的 operator[] 在返回一个引用前，先执行边界校验、打印日志、校验数据完整性。把所有这些同时放进 const 和 non-const operator[]中，就会导致代码存在一定的重复：

```c++
 class MyString
 {
 public:
     const char& operator[](std::size_t position) const 
     { 
         ...    // 边界检查
         ...    // 日志记录
         ...    // 校验数据完整性
         return text[position]; 
    }

    char& operator[](std::size_t position)
    { 
        ...    // 边界检查
        ...    // 日志记录
        ...    // 校验数据完整性
        return text[position]; 
    }
private:
    std::string text;
};
```

可以有一种解决方法，避免代码的重复：

```c++
class MyString
{
public:
    const char& operator[](std::size_t position) const  // 一如既往
    {
        ...    // 边界检查
        ...    // 日志记录
        ...    // 校验数据完整性
        return text[position];
    }

    char& operator[](std::size_t position)
    {
        return const_cast<char&>(
                static_cast<const MyString&>(*this)[position]
                );
    }
private:
    std::string text;
};
```

这份代码有两个转型动作：

- static_cast(*this)[position]，表示将 MyString& 转换成 const MyString&，**可让其调用 const operator[] 兄弟**；
- const_cast<char&>( … )，表示将 const char & 转换为 char &，**让其是 non-const operator[] 的返回类型**。

虽然语法有一点点奇特，但「运用 const 成员函数实现 non-const 孪生兄弟 」的技术是值得了解的。

需要注意的是：我们可以在 non-const 成员函数调用 const 成员函数，但是不可以反过来，在 const 成员函数调用 non-const 成员函数调用，原因是对象有可能因此改动，这会违背了 const 的本意。

------

> **细节 02 小结 - 请记住**将某些东西声明为 const 可帮助编译器探测出错误用法。const 可以被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体。当 const 和 non-const 成员函数有着实质等价的实现时，令 non-const 版本调用 const 版本可避免代码重复。

------



# 细节 03：确定对象被使用前先被初始化

> 内置类型初始化

如果你这么写：

```c++
int x;
```

在某些语境下 x 保证被初始化为 0，但在其他语境中却不保证。那么可能在读取未初始化的值会导致不明确的行为。

为了避免不确定的问题，最佳的处理方法就是：永远在使用对象之前将它初始化。 例如：

```c++
int x = 0;                    // 对 int 进行手工初始化
const char* text = "abc";     // 对指针进行手工初始化
```

------

> 构造函数初始化

对于内置类型以外的任何其他东西，初始化责任落在**构造函数**。

规则很简单：确保每一个构造函数都将对象的**每一个成员初始化**。但是别混淆了**赋值**和**初始化**。

考虑用一个表现学生的class，其构造函数如下：

```c++
class Student {
public:
    Student(int id, const std::string& name, const std::vector<int>& score)
    {
        m_Id = id;          // 这些都是赋值
        m_Name = name;      // 而非初始化
        m_Score = score;
    }
private:
    int m_Id;
    std::string m_Name;
    std::vector<int> m_Score;
};
```

上面的做法并非初始化，而是赋值，这不是最佳的做法。因为 **C++ 规定，对象的成员变量的初始化动作发生在进入构造函数本体之前，在构造函数内，都不算是被初始化，而是被赋值**。

初始化的写法是使用成员初值列，如下：

```c++
    Student(int id,
            const std::string &name,
            const std::vector<int> &score)
            : m_Id(id),
              m_Name(name),  // 现在，这些都是初始化
              m_Score(score) 
     {}                      //  现在，构造函数本体不必有任何动作
```

这个构造函数和上一个构造函数的最终结果是一样的，但是效率较高，凸显在：

- 上一个构造函数（赋值版本）首先会先自动调用 m_Name 和 m_Score 对象的**默认构造函数**作为初值，然后在构造函数体内立刻再对它们进行**赋值操作**，这期间经历了**两个**步骤。
- 这个构造函数（成员初值列）避免了这个问题，**只会发生了一次复制构造函数**，本例中的 m_Name 以 name 为初值进行复制构造，m_Score 以 score 为初值进行复制构造。

另外一个注意的是初始化次序（顺序）：

1. 先是基类对象，再初始化派生类对象（如果存在继承关系）；
2. 在类里成员变量总是以声明次序被初始化，如本例中 m_Id 先被初始化，再是 m_Name，最后是 m_Score，否则会出现编译出错。

------

> 避免「跨编译单元之初始化次序」的问题

现在，我们关系的问题涉及至少两个以上源码文件，每一个内含至少一个 non-local static 对象。

存在的问题是：如果有一个 non-local static 对象需要等另外一个 non-local static 对象初始化后，才可正常使用，那么这里就需要**保证次序**的问题。

下面提供一个例子来对此理解：

```c++
class FileSystem
{
public:
    ...
    std::size_t numDisk() const; // 众多成员函数之一
    ...
};

extern FileSystem tfs; // 预备给其他程序员使用对象
```

现假设另外一个程序员建立一个class 用以处理文件系统内的目录，很自然他们会用上 tfs 对象：

```c++
class Directory
{
public:
    Directory( params )
    {
        std::size_t disks = tfs.numDisk(); // 使用 tfs 对象
    }
    ...
};
```

使用 Directory 对象：

```c++
Directory tempDir( params );
```

那么现在，初始化次序的重要性凸显出来了，除非 tfs 对象在 tempDir 对象之前被初始化，否则 tempDir 的构造函数会用到尚未初始化的 tfs， 就会出现**未定义的现象**。

由于 C++ 对「定义于不同的编译单元内的 non-local static 对象」的初始化相对次序并无明确定义。但我们可以通过一个小小的设计，解决这个问题。

唯一需要做的是：将每个 non-local static 对象搬到自己的专属函数内（该对象在此函数内被声明为 static），这些函数返回一个引用指向它所含的对象。

没错也就是**单例模式**，代码如下：

```c++
class FileSystem
{
public:
    ...
    static FileSystem& getTfs() // 该函数作用是获取 tfs 对象，
    {
        static FileSystem tfs;  // 定义并初始化一个 local static 对象,
        return tfs;             // 返回一个引用指向上述对象。
    }
    ...
};


class Directory
{
public:
   ...
    Directory( params )
    {
        std::size_t disks = FileSystem::getTfs().numDisk(); // 使用 tfs 对象
    }
    ...
};
```

这么修改后，Directory 构造函数就会先初始化 tfs 对象，就可以避免次序问题了。虽然内含了 static 对象，但是在 C++11 以上是线程安全的。

------

> **细节 03 小结 - 请记住**为内置类型进行手工初始化，因为 C++ 不保证初始化它们。构造函数最好使用成员初值列，而不要在构造函数本体内使用赋值操作。初值列列出的成员变量，其排列次序应该和它们在 class 中的声明次序（顺序）相同。为避免“跨编译单元之初始化次序”的问题，请以 local static 对象替代 non-local static 对象。



# 细节 04：了解 C++ 默默编写并调用哪些函数

当你写了如下的空类：

```c++
class Student { };
```

编译器就会它声明，并且这些函数都是 public 且 inline：

1. 复制构造函数
2. 赋值操作符函数
3. 析构函数
4. 默认构造函数（如果没有声明任何构造函数）

就好像你写下这样的代码：

```c++
class Student 
{ 
    Student() { ... }                              // 默认构造函数
    Student(const Student& rhs) { ... }            // 复制构造函数
    Student& operator=(const Student& rhs) { ... } // 赋值操作符函数
    ~Student() { ... }                             // 析构函数
};
```

**唯有当这些函数被需要调用时，它们才会被编译器创建出来**，下面代码造成上述每一个函数被编译器产出：

```c++
Student stu1;         // 默认构造函数
                      // 析构函数
Student stu2(stu1);   // 复制构造函数
stu2 = stu1;          // 赋值操作符函数
```

编译器为我们写的函数，来说说这些函数做了什么？

- 默认构造函数和析构函数主要是给编译器一个地方用来放置隐藏幕后的代码，像是调用基类和非静态成员变量的构造函数和析构函数。注意，编译器产出的析构函数是个 non-virtual，除非这个 class 的 base class 自身声明有 virtual 析构函数。
- 复制构造函数和赋值操作符函数，编译器创建的版本只是单纯地将来源对象的每一个非静态成员变量拷贝到目标对象。

------

> 编译器拒绝为 class 生出 operator= 的情况

对于赋值操作符函数，只有当生出的代码合法且有适当机会证明它有意义，才会生出 operator= ，若万一两个条件有一个不符合，则编译器会**拒绝**为 class 生出 operator= 。

举个例子：

```c++
template<class T>
class Student
{
public:
    Student(std::string & name, const T& id); // 构造函数
    ...                          // 假设未声明 operator=
priavte:
    std::string& m_Name;    // 引用
    const T m_Id; // const
};
```

现考虑下面会发生什么：

```c++
std::string name1("小美");
std::string name2("小林");

Student<int> p(name1, 1);
Student<int> s(name2, 2);

p = s;            // 现在 p 的成员变量会发生什么？
```

赋值之前， p.m_Name 和 s.m_Name 都指向 string 对象且不是同一个。赋值之后 p.m_Name 应该指向 s.m_Name 所指的那个 string 吗？也就是说引用自身可被改动吗？如果是，那就开辟了新天地，**因为 C++ 并不允许「让引用更改指向不同对象」**。

面对这个难题，C++ 的响应是拒绝编译那一行赋值动作，本例子拒绝生成的 operator= 原因如下：

- 如果你需要在一个「内含引用的成员」（如本例的 m_Name ）的class 内支持赋值操作，你必须自己定义赋值操作函数，这种情况是编译器不会为你自动生成赋值操作函数的。
- 还有面对「内含 const 成员」（如本例的 m_Id ）的class，编译器也是会拒绝生成 operator=，因为更改 const 成员是不合法的。

最后还有一个情况：如果某个基类将 operator= 函数声明为 private ，编译器将拒绝为其派生类生成 operator= 函数。

------

> **细节 04 小结 - 请记住**编译器可以暗自为 class 创建默认构造函数（如果没有声明任何构造函数）、复制构造函数、赋值操作符函数，以及析构函数。编译器拒绝为 class 创建 operator= 函数情况：（1） 内含引用的成员、（2） 内含 const 的成员、（3）基类将 operator= 函数声明为 private。

------



# 细节 05：若不想使用编译器自动生成的函数，就该明确拒绝

在不允许存在一模一样的两个对象的情况下，可以把**复制构造函数和赋值操作符函数**声明为 private，这样既可防止编译器自动生成这两个函数。如下例子：

```c++
class Student
{
public:
    ...
private:
    ... 
    Student(const Student&);             // 只有声明
    Student& operator=(const Student&);  // 只有声明
};
```

这样的话，Student 对象就无法操作下面的情况了：

```c++
Student stu1;
Student stu2(stu1);   // 错误，禁用了 复制构造函数

stu2 = stu1;          // 错误，禁用了 赋值操作符函数
```

更容易扩展的解决方式是，可以专门写一个为阻止 copying 动作的基类：

```c++
class Uncopyale
{
protect:              // 允许派生类对象构造和析构
    Uncopyale() {}                
    ~Uncopyale() {}
private:             // 禁止派生类对象copying
    Uncopyale(const Uncopyale&);
    Uncopyale& operater=(const Uncopyale&);
};
```

使用方式很简单，只需要 private 形式的继承：

```c++
class Student : private Uncopyale{  
    ...  // 派生类不用再声明复制构造函数和赋值操作符函数
};
```

那么只要某个类需要禁止 copying 动作，则只需要 private 形式的继承 Uncopyale 基类即可。

------

> **细节 05 小结 - 请记住**如果不想编译器自动生成函数，可将相应的成员函数声明为 private 并且不予实现。使用像 Uncopyale 这样的基类也是一种做法。



# 细节 06：为多态基类声明 virtual 析构函数

多态特性的基础内容，可移步到我的旧文进行学习 --> 掌握了多态的特性，写英雄联盟的代码更少啦！

> 多态性质基类需声明 virtual 析构函数

如果在多态性质的基类，没有声明一个 virtual 析构函数，那么在 delete 基类指针对象的时候，只会调用基类的析构函数，而不会调用派生类的析构函数，这就是存在了**泄漏内存和其他资源的情况**。

如下有多态性质基类，没有声明一个 virtual 析构函数的例子：

```c++
// 基类
class A
{
public:
    A()  // 构造函数
    {
        cout << "construct A" << endl;
    }

    ~A() // 析构函数
    {
        cout << "Destructor A" << endl;
    }
};

// 派生类
class B : public A
{
public:
    B()  // 构造函数
    {
        cout << "construct B" << endl;
    }

    ~B()// 析构函数
    {
        cout << "Destructor B" << endl;
    }
};

int main()
{
    A *pa = new B();
    delete pa;    // 释放资源

    return 0;
}
```

输出结果：

```
construct A
construct B
Destructor A 
```

从上面的结果，是发现了在 delete 基本对象指针时，没有调用派生类 B 的析构函数。问题出在 pa 指针指向派生类对象，而那个对象却经由一个基类指针删除，而目前的基类没有 virtual 析构函数。

消除这个问题的做法很简单：为了避免泄漏内存和其他资源，需要把基类的析构函数声明为 virtual 析构函数。改进如下：

```c++
 // 基类
class A
{
public:
    ....            // 如上
    virtual ~A()   // virtual 析构函数
    {
        cout << "Destructor A" << endl;
    }
};
...                // 如上
```

此后删除派生类对象就会如你想要的那般，是的，它会销毁整个对象，包括所有派生类成份。

> 非多态性质基类无需声明 virtual 函数

<font color="red">当类的设计目的不是被当做基类，令其析构函数为 virtual 往往是个**馊主意**。</font>

若类里声明了 virtual 函数，对象必须携带某些信息。主要用来运行期间决定哪一个 virtual 函数被调用。

*这份信息通常是由一个所谓 vptr（virtual table pointer —— 虚函数表指针）指针指出。vptr 指向一个由函数指针构成的数组，称为 vtbl（virtual table —— 虚函数表）；每一个带有 virtual 函数的类都有一个相应的 vtbl。当对象调用某一 virtual 函数，实际被调用的函数取决于该对象的 vptr 所指向的那个 vtbl，接着编译器在其中寻找适当的函数指针，从而调用对应类的函数。*

既然内含 virtual 函数的类的对象必须会携带信息，那么必然其对象的体积是会增加的。

- 在 32 位计算机体系结构中将多占用 4个字节（存放 vptr ）；
- 在 64 位计算机体系结构则将多占用 8 个字节（存放 vptr ）。

因此，无端地将所有类的析构函数声明为 virtual ，是错误的，原因是会增加不必要的体积。

**许多人的心得是：只有当 class 内含至少一个 virtual 函数，才为它声明 virtual 析构函数。**

------

> **细节 06 小结 - 请记住**在多态性质的基类，应该声明一个 virtual 析构函数。如果 class 带有任何 virtual 函数，它就应该拥有一个 virtual 析构函数。类的设计目的如果不是为基类使用的，或不是为了具备多态性，就不该声明 virtual 析构函数。

------



# 细节 07：绝不在构造和析构过程中调用 virtual 函数

我们不该在构造函数和析构函数体内调用 virtual 函数，因为这样的调用不会带来你预想的结果。

我们看如下的代码例子，来说明：

```c++
 // 基类
class CFather
{
public:
    CFather()
    {
        hello();
    }

    virtual ~CFather()
    {
        bye();
    }

    virtual void hello() // 虚函数
    {
        cout<<"hello from father"<<endl;
    }

    virtual void bye() // 虚函数
    {
        cout<<"bye from father"<<endl;
    }
};

// 派生类
class CSon : public CFather
{
public:
    CSon() // 构造函数
    {
        hello();
    }

    ~CSon()  // 析构函数
    {
        bye();
    }

    virtual void hello() // 虚函数
    {
        cout<<"hello from son"<<endl;
    }

    virtual void bye() // 虚函数
    {
        cout<<"bye from son"<<endl;
    }
};
```

现在，当以下这行被执行时，会发生什么事情：

```c++
CSon son;
```

先列出输出结果：

```c++
hello from father
hello from son
bye from son
bye from father
```

无疑地会有一个 CSon（派生类） 构造函数被调用，但首先 CFather（基类） 构造函数一定会更早被调用。 CFather（基类） 构造函数体里调用 virtual 函数 hello，这正是引发惊奇的起点。这时候被调用的 hello 是 CFather 内的版本，而不是 CSon 内的版本。

说明，**基类构造期间 virtual 函数绝不会下降到派生类阶层**。取而代之的是，对象的作为就像隶属于基类类型一样。

**非正式的说法或许比较传神：<font color="red">在基类构造期间，virtual 函数不是 virtual 函数。</font>**

相同的道理，也适用于析构函数。

------

> **细节 07 小结 - 请记住**在构造和析构期间不要调用 virtual，因为这类调用不会下降至派生类。

------



# 细节 08：令 operator= 返回一个 reference to *this

关于赋值，又去的是你可以把它们写成连锁形式：

```c++
int x, y, z;
x = y = z = 15;  // 赋值连锁形式 
```

同样有趣的是，赋值采用右结合律，所以上述连锁赋值被解析为：

```c++
x = (y = ( z = 15 ));
```

这里 15 先被赋值给 z，然后其结果再被赋值给 y，然后其结果再赋值给 x 。

为了实现「连锁赋值」，赋值操作必须返回一个 reference （引用）指向操作符的左侧实参。这是我们为 classes 实现赋值操作符时应该遵循的协议：

```c++
class A
{
public:
...
    A& operator=(const A& rhs) // 返回类型是一个引用，指向当前对象。
    {
        ...
        return *this;           // 返回左侧对象
    }
...
};
```

这个协议不仅适用于以上标准赋值形式，也适用于所有赋值相关运算（+=, -=, *=, 等等），例如：

```c++
class A
{
public:
...
    A& operator+=(const A& rhs) // 这个协议适用于 +=, -=, *=, 等等。
    {
        ...
        return *this;
    }
...
};
```

注意，这只是个协议，并无强制性。如果不遵循它，代码一样可以通过编译，但是会**破坏原本的编程习惯。**

------

> **细节 08 小结 - 请记住**令赋值操作符返回一个 reference to *this。

------



# 细节 09：在 operator= 中处理「自我赋值」

「自我赋值」发生在对象被赋值给自己时：

```c++
class A { ... };
A a;
...
a = a;   // 赋值给自己
```

这看起来有点愚蠢，但它合法，所以不要认定我们自己绝对不会那么做。

此外赋值动作并不总是那么一眼被识别出来，例如：

```c++
a[i] = a[j]; // 潜在的自我赋值
```

如果 i 和 j 有相同的值，这便是个自我赋值。再看：

```c++
*px = *py;  // 潜在自我赋值
```

如果 px 和 py 刚好指向同一个东西，这也是自我赋值，这些都是并不明显的自我赋值。

考虑到我们的类内含指针成员变量：

```c++
class B { ... };
class A
{
...
private:
    B * pb; // 指针，指向一个从堆分配而得的对象
}
```

下面是operator = 实现代码，表面上看起来合理，但自我赋值出现时并不安全：

```c++
A& A::operator=(const A& rhs) // 一份不安全的operator = 实现版本
{
    delete pb;             // 释放旧的指针对象
    pb = new B(*rhs.pb);  // 生成新的地址
    return *this;
}
```

这里的自我赋值的问题是， operator= 函数内的 *this（赋值的目的端）和 rhs 有可能是**同一个对象**。果真如此 delete 就不只是销毁当前对象的 pb，它也销毁 rhs 的 pb。

相当于发生了自我销毁（自爆/自灭）过程，那么此时 A 类对象持有了一个指向一个被销毁的 B 类对象。非常的危险，请勿模仿！

下面来说说如何规避这种问题的方式。

------

> 方式一：比较来源对象和目标对象的地址

要想阻止这种错误，传统的做法是在 operator= 函数最前面加一个 if 判断，判断是否是自己，不是才进行赋值操作：

```c++
A& A::operator=(const A& rhs) 
{
    if(this == &rhs) 
       return *this;    // 如果是自我赋值，则不做任何事情。

    delete pb;             // 释放旧的指针对象
    pb = new B(*rhs.pb);   // 生成新的地址
    return *this;
}
```

这样错虽然行得通，但是不具备自我赋值的安全性，也不具备异常安全性：

- 如果「 new B 」这句发生了异常（申请堆内存失败的情况），A 最终会持有一个指针指向一块被删除的 B，这样的指针是有害的。

我旧文里《C++ 赋值运算符'='的重载（浅拷贝、深拷贝）》在规避这个问题试，就采用的是方式 一，这个方式是不合适的。

------

> 方式二：精心周到的语句顺序

把代码的顺序重新编排以下就可以避免此问题，例如一下代码，我们只需之一在赋值 pb 所指东西之前别删掉 pb :

```c++
A& A::operator=(const A& rhs) 
{
    A* pOrig = pb;       // 记住原先的pb
    pb = new B(*rhs.pb); // 令 pb 指向 *pb的一个副本
    delete pOrig;        // 删除原先的pb
    return *this;
}
```

现在，如果「 new B 」这句发生了异常，pb 依然保持原状。即使没有加 if 自我判断，这段代码还是能够处理自我赋值，因为我们对原 B 做了一份副本、删除原 B 、然后返回引用指向新创造的那个副本。

它或许不是处理自我赋值的最高效的方法，但它行得通。

------

> 方式三：copy and swap

更高效的方式使用所谓的 copy and swap 技术，实现方法如下：

```c++
 class A
{
...
void swap(A& rhs) // 交换*this 和 rhs 的数据
{
    using std::swap;
    swap(pb, rhs.pb);
}
...
private:
    B * pb; // 指针，指向一个从堆分配而得的对象
}
};

A& A::operator=(const A& rhs)
{
    A temp(rhs); // 为 rhs 制作一份复件（副本）
    swap(tmp);   // 将 *this 数据和上述复件的数据交换。
    return *this;
}
```

当类里 operator= 函数被声明为「以 by value 方式接受实参」，那么由于 by value 方式传递东西会造成一份复件（副本），则直接 swap 交换即可，如下：

```c++
A& A::operator=(A rhs) // rhs是被传对象的一份复件
{
    swap(rhs);        // 将 *this 数据和复件的数据交换。
    return *this;
}
```

------

> **细节 09 小结 - 请记住**确保当对象自我赋值时，operator= 有良好行为。其中技术包括比较来源对象和目标对象的地址、精心周到的语句顺序、以及 copy-and-swap。确保任何函数如果操作一个以上的对象，而其中多个对象是同个对象时，其行为忍然正常。

------



# 细节 10：复制对象时勿忘其每一个成分

在以下我把**复制构造函数和赋值操作符函数**，称为「copying 函数」。

如果你声明自己的 copying 函数，那么编译器就不会创建默认的 copying 函数。但是，当你在实现 copying 函数，遗漏了某个成分没被 copying，编译器却不会告诉你。

> 确保对象内的所有成员变量 copying

考虑用一个 class 用来表示学生，其中自实现 copying 函数，如下:

```c++
class Student
{
public:
    ...
    Student(const Student& rhs);
    Student& operator=(const Student& rhs);
    ...
private:
    std:: string name;
}

Student::Student(const Student& rhs)
  : name(rhs.name)   // 复制 rhs 的数据
{  }

Student& Student::operator=(const Student& rhs)
{
    name = rhs.name; // 复制 rhs 的数据
    return *this;
}
```

这里的每一件事情看起来都很好，直到另一个成员变量加入战局：

```c++
class Student
{
public:
    ... // 同前
private:
    std::string name;
    int score;
}
```

这时候遗漏对新成员变量的 copying。大多数编译器对此不做任何报错。

**结论很明显：如果你为 class 添加一个成员变量，你必须同时修改 copying 函数。**

------

> 确保所有 base class （基类） 成分 copying

一旦存在继承关系的类，可能会造成此一主题最黑暗肆意的一个潜在危机。试考虑：

```c++
class CollegeStudent : public Student // 继承 Student
{
public:
...
    CollegeStudent(const CollegeStudent& rhs);
    CollegeStudent& operator=(const CollegeStudent& rhs);
...
private:
    std::string major;
};

CollegeStudent::CollegeStudent(const CollegeStudent& rhs)
 : major(rhs.major)
{ }

CollegeStudent& CollegeStudent::operator=(const CollegeStudent& rhs)
{
    major = rhs.major;
    return *this;
}
```

CollegeStudent 的 copying 函数看起来好像复制了 CollegeStudent 内的每一样东西，但是请再看一眼。是的，它们复制了 CollegeStudent 声明的成员变量，但每个 CollegeStudent 还内含所继承的 Student 成员变量复件（副本)，而哪些成员变量却未被复制。

所以任何时候只要我们承担起「为派生类撰写 copying 函数」的重则大任，必须很小心地也复制其 base class 成分：

```c++
 CollegeStudent::CollegeStudent(const CollegeStudent& rhs)
  : Student(rhs),  // 调用 base class 的 copy构造函数
    major(rhs.major)
 { }
 
 CollegeStudent& CollegeStudent::operator=(const CollegeStudent& rhs)
 {
     Student::operator=(rhs); // 对 base class 成分进行赋值动作
     major = rhs.major;
    return *this;
}
```

**所以我们不仅要确保复制所有类里的成员变量，还要调用所有 base classes 内的适当的 copying 函数。**

------

> 消除 copying 函数之间的重复代码

还要一点需要注意的：不要令复制「构造函数」调用「赋值操作符函数」，来减少代码的重复。这么做也是存在危险的，假设调用赋值操作符函数不是你期望的。—— 错误行为。

同样也不要令「赋值操作符函数」调用「构造函数」。

如果你发现你的「复制构造函数和赋值操作符函数」有近似的代码，消除重复代码的做法是：**建立一个新的成员函数给两者调用**。

------

> 细节 10 小结 - 请记住Copying 函数（复制构造函数和赋值操作符函数）应该确保复制「对象内的所有成员变量」及「所有 base class（基类） 成分」。不要尝试以某个 copying 函数实现另外一个 coping 函数。应该将共同地方放进第三个函数中，并由两个 copying 函数共同调用。
