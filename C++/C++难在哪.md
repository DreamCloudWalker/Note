参考https://www.zhihu.com/question/536791750/answer/3033320923?utm_campaign=shareopn&utm_medium=social&utm_psn=1842119477883437056&utm_source=wechat_session

作者：码农出击
链接：https://www.zhihu.com/question/536791750/answer/3033320923
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



我们只谈构造函数。假如我们有一个类Teacher。

```c++
class Teacher
{
private:
    std::string name;
    std::string position;
};
```


我们考虑给Teacher类加上构造函数。

```c++
class Teacher
{
private:
    std::string name;
    std::string position;

public:
    Teacher(const std::string& n, const std::string& p)
        : name(n), position(p) {}
};
```


虽然语义正确，但是如果我们的实参只为了传递给Teacher，传递之后而没有其他作用的话，那么这个实现是效率低下的。字符串的拷贝花销可观（关于`std::string`的COW，SSO，view的讨论是另一个故事了）。我们在C++11里面有**右值引用**和**move语义**，所以呢，我们可以改成这样。

```c++
class Teacher
{
private:
    std::string name;
    std::string position;

public:
    Teacher(const std::string& n, const std::string& p)
        : name(n), position(p) {}

    Teacher(std::string&& n, std::string&& p)
        : name(std::move(n)), position(std::move(p)) {};
};
```


你可能觉得这样也已经不错了。不过我们还有可能第一个参数右值，第二个参数左值。或者第一个参数左值，第二个参数右值。所以实际上我们需要四个函数的重载。

```c++
class Teacher
{
private:
    std::string name;
    std::string position;

public:
    Teacher(const std::string& n, const std::string& p)
        : name(n), position(p) {}

    Teacher(std::string&& n, std::string&& p)
        : name(std::move(n)), position(std::move(p)) {};

    Teacher(const std::string&& n, const std::string& p)
        : name(std::move(n)), position(p) {}

    Teacher(const std::string& n, const std::string&& p)
        : name(n), position(std::move(p)) {}
};
```


代码有点多。我们有没有什么方法写一个通用的函数来实现这四个函数呢？有。我们在C++11中有**完美转发**。

```c++
class Teacher
{
private:
    std::string name;
    std::string position;

public:
    template <typename S1, typename S2>
    Teacher(S1&& n, S2&& p)
        : name(std::forward<S1>(n)), position(std::forward<S2>(p)) {};
};
```


完成了。美滋滋。然而事情没有这么简单。如果我们的position有默认值，然后我们写如下代码的话。

```c++
class Teacher
{
private:
    std::string name;
    std::string position;

public:
    template <typename S1, typename S2 = std::string>
    Teacher(S1&& n, S2&& p = "lecturer")
        : name(std::forward<S1>(n)), position(std::forward<S2>(p)) {};
};


int main()
{
    Teacher t1 = { "david", "assistant" };
    Teacher t2{ t1 };
}
```

我们出现了编译期错误。因为`Teacher t2{ t1 };`的**重载决议**的最佳匹配是我们的模板，而不是默认的拷贝构造函数，因为拷贝构造函数要求t1是const的。所以，我们可能需要**SFINAE**和**type traits**来修改我们的代码。注意，默认函数参数不能**类型推导**，所以我们才需要的S2的默认模板参数。

```c++
class Teacher
{
private:
    std::string name;
    std::string position;

public:
    template <typename S1, typename S2 = std::string,
    typename = std::enable_if_t<!std::is_same_v<S1, Teacher>>>
    Teacher(S1&& n, S2&& p = "lecturer")
        : name(std::forward<S1>(n)), position(std::forward<S2>(p)) {};
};
```


仍然不对哦，因为我们的完美转发有**引用折叠**机制，我们应该判断的S1是Teacher&而不是Teacher。其次，如果有类继承我们的Teacher的话，拷贝的时候依然会出现这个问题，所以我们需要的不是`is_same`而是`is_convertible`。然而，如果我们直接写`std::is_convertible_v<S1, Teacher>`的话，我们实际上判定是不是可以转换的时候，还是会去看我们的构造函数。也就是说我们自己依赖了自己，无穷递归。所以我们需要的是`std::is_convertible_v<S1, std::string>`。所以，我们修改我们的代码。

```c++
class Teacher
{
private:
    std::string name;
    std::string position;

public:
    template <typename S1, typename S2 = std::string,
    typename = std::enable_if_t<std::is_convertible_v<S1, std::string>>>
    Teacher(S1&& n, S2&& p = "lecturer")
        : name(std::forward<S1>(n)), position(std::forward<S2>(p)) {};
};
```


其次，因为我们的默认参数是字面量，字面量是`const char[]`类型的。我们调用构造函数的时候，也会用字面量。字面量不是`std::string`类型会造成很多问题。然而在C++14中，我们可以用**User-defined literals**来把字面量声明成`std::string`类型的。不过记得命名空间，这个名字空间不在std中。我们这里不再讨论了。

我们接下来讨论用构造函数初始化的问题。初始化有很多种写法，以下我列出有限的几种。

```c++
Teacher t1("david"s);
Teacher t2 = Teacher("david"s);

Teacher t3{ "lily"s };
Teacher t4 = { "lily"s };
Teacher t5 = Teacher{ "lily"s };

auto t6 = Teacher("david"s);
auto t7 = Teacher{ "lily"s };
```


我们用了**auto**。然而auto是**decay**的。而**decltype(auto)**不。所以，以下代码如果用auto的话可能不是你需要的。

```c++
const Teacher& t8 = t1; 
auto t10 = t8;
```

我们需要写`const auto&`。

此外，我们可以看出，用小括号和大括号好像没什么区别。不过，在一些情况下会有很大的差别。我们列出一些。

```c++
std::vector<int> vec1(30, 5); std::vector<int> vec2{ 30, 5 };
```

甚至因为C++17的**构造函数自动推导**，我们可以写出更加疯狂的代码。

```c++
std::vector vec3{ vec1.begin(), vec2.end() };
```

这个代码是用初始化列表初始化的，也就是说我们得到的vec3中有两个iterator。

好了，我们回过头来说auto。我们可以看到好像我们所有的初始化都可以用auto。是这样吗？如果我们写atomic的代码呢？

```c++
auto x = std::atomic<int>{ 10 };
```

是可以的。因为在C++17中我们有**Copy Elision**。所以这里没有拷贝函数的调用，和直接定义并初始化是一致的。但是atomic初始化是有问题的。

```c++
std::atomic<int> x{};
```

这样是不能零初始化的。当然了，这显然是API的不一致，或者说错误。所以我们有**LWG issue 2334**。预计在C++20修复这个问题。嘻嘻。

以上内容基于《C++ templates》的作者Nicolai Josuttis的几场talk。

> 作者：琉璃 
> 链接：https://www.zhihu.com/question/30196513/answer/563560938 
> 来源：知乎

这里详细谈下C++ 的学习路线，按照这个路线去学习C++，每个阶段都帮你规划好了学习时间，只要你努力且认真的去学了， 保证帮你既高效又扎实的学好C++：



**一、C++基础（3个月）**

1、面向对象的三大特性：封装、继承、多态
2、类的访问权限：private、protected、public
3、类的构造函数、析构函数、赋值函数、拷贝函数
4、移动构造函数与拷贝构造函数对比
5、深拷贝与浅拷贝的区别
6、空类有哪些函数？空类的大小？
7、内存分区：全局区、堆区、栈区、常量区、代码区
8、C++与C的区别
9、struct与class的区别
10、struct内存对齐
11、new/delete与malloc/free的区别
12、内存泄露的情况
13、sizeof与strlen对比
14、指针与引用的区别
15、野指针产生与避免
16、多态：动态多态、静态多态
17、虚函数实现动态多态的原理、虚函数与纯虚函数的区别
18、继承时，父类的析构函数是否为虚函数？构造函数能不能为虚函数？为什么？
19、静态多态：重写、重载、模板
20、static关键字：修饰局部变量、全局变量、类中成员变量、类中成员函数
21、const关键字：修饰变量、指针、类对象、类中成员函数
22、extern关键字：修饰全局变量
23、volatile关键字：避免编译器指令优化
24、四种类型转换：static_cast、dynamic_cast、const_cast、reinterpret_cast
25、右值引用
26、std::move函数
27、四种智能指针及底层实现：auto_ptr、[unique_ptr](https://zhida.zhihu.com/search?content_id=581597711&content_type=Answer&match_order=1&q=unique_ptr&zhida_source=entity)、shared_ptr、weak_ptr
28、shared_ptr中的循环引用怎么解决？（weak_ptr）
29、vector与list比较
30、vector迭代器失效的情况
31、map与unordered_map对比
32、set与unordered_set对比
33、STL容器空间配置器

**参考书籍：**《C++ Primer》（第5版）、《STL源码剖析》、《深度探索C++对象模型》

**下载地址：**

- 链接：[https://pan.baidu.com/s/1qqAR6iqjur1sfmzeZjcrwg](https://link.zhihu.com/?target=https%3A//pan.baidu.com/s/1qqAR6iqjur1sfmzeZjcrwg)
- 提取码：m6gx