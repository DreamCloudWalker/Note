## C++ 理解 std::forward 完美转发

C++中存在两个比较难理解的操作：

1. `std::move` : 移除左值属性。
2. `std::forward` : 完美转发。

下面来讲一讲`std::forward`的用途和转换原理。



## 1. 背景

假如我们封装了一个操作，主要是用来创建对象使用（类似设计模式中的工厂模式），这个操作如下：

1. 可以接受不同类型的参数，然后构造一个对象的指针。
2. 性能尽可能高。

现在假设这个类的定义如下：

```cpp
class CData
{
public:
    CData() = delete;
    CData(const char* ch) : data(ch)
    {
        std::cout << "CData(const char* ch)" << std::endl;
    }
    CData(const std::string& str) : data(str) 
    {
        std::cout << "CData(const std::string& str)" << std::endl;
    }
    CData(std::string&& str) : data(str)
    {
        std::cout << "CData(std::string&& str)" << std::endl;
    }
    ~CData()
    {
        std::cout << "~CData()" << std::endl;
    }
private:
    std::string data;
};
```

**这里如果需要高效率，对于右值的调用应该使用CData(std::string&& str) : data(str)移动函数操作。**

### 1.1 普通模板定义

如果只要一个函数入口来创建对象，那么使用模板是不错的选择，例如可以如下：

```cpp
template<typename T>
CData* Creator(T t)
{
    return new CData(t);
}
void Forward()
{
    const char* value = "hello";
    std::string str1 = "hello";
    std::string str2 = " world";
    //CData* p = Creator(value);
    CData* p = Creator(str1);
    //CData* p = Creator(str1 + str2);

    delete p;
}
```

这种办法虽然行得通，但是比较挫，因为每次调用`Creator(T t)`的时候，都需要拷贝内存，明显不满足高效的情况。

### 1.2 引用模板定义

上面说的值拷贝不能满足内容，那么我们使用引用就可以解决问题了吧？

```cpp
template<typename T>
CData* Creator(T& t)
{
    return new CData(t);
}
void Forward()
{
    const char* value = "hello";
    std::string str1 = "hello";
    std::string str2 = " world";
    CData* p = Creator(value);
    //CData* p = Creator(str1);

    delete p;
}
```

但是这种情况对于`CData* p = Creator(str1 + str2)`无法解决问题，因为右值无法赋值到左值的引用。

### 1.3 右值引用模板

从上面我们比较容易想到，使用&&来解决拷贝的问题。

```cpp
template<typename T>
CData* Creator(T&& t)
{
    return new CData(t);
}
void Forward()
{
    const char* value = "hello";
    std::string str1 = "hello";
    std::string str2 = " world";
    CData* p = Creator(str1 + str2);

    delete p;
}
```

由于模板中引用折叠的规则：

1. T& & –> T&。
2. T&& & –> T&。
3. T& && –>T&。
4. T&& && –> T&&。

这种使用基本能够满足使用了。但是真的吗？别急我们来分析一下效率问题。

对于`CData* p = Creator(str1 + str2);`的调用：

1. 产生一个右值`str1 + str2`.
2. `CData* Creator(T&& t)`右值引用，此时为：`std::string&& t`。
3. 那么`return new CData(t);`为右值引用，调用构造函数的`CData(std::string&& str) : data(str)`移动构造函数。

但是是否是这样的呢？如果是这样效率确实是最佳的，因为我们只需要右值直接移动就是最高效率了，运行结果如下：

```cpp
CData(const std::string& str)
~CData()
```

显然并没有调用移动函数，原因是因为在函数：

```cpp
CData* Creator(T&& t)
{
    return new CData(t);
}
```

t 是一个变量，为左值，**无论左值引用类型的变量还是右值引用类型的变量，都是左值，因为它们有名字。**，例如可以写如下代码：

```cpp
int a = 100;
int&& b = 100;
int& c = b; //正确，b为左值
int&d = 100; //错误
```



## 2. forward完美转发

我们使用fordward来完美解决这个问题。

### 2.1 完美转发模板

```cpp
template<typename T>
CData* Creator(T&& t)
{
    return new CData(std::forward<T>(t));
}

void Forward()
{
    const char* value = "hello";
    std::string str1 = "hello";
    std::string str2 = " world";
    CData* p = Creator(str1 + str2);

    delete p;
}
```

此时运行结果为：

```cpp
CData(std::string&& str)
~CData()
```

### 2.2 结论

所谓的完美转发，是指**std::forward会将输入的参数原封不动地传递到下一个函数中，这个“原封不动”指的是，如果输入的参数是左值，那么传递给下一个函数的参数的也是左值；如果输入的参数是右值，那么传递给下一个函数的参数的也是右值。**

这个对于上面一个例子带来的好处就是函数转发仍旧为右值引用，可以使用移动函数。

### 2.3 原理

`std::forward`定义如下：

```cpp
template<class _Ty>
_NODISCARD constexpr _Ty&& forward(remove_reference_t<_Ty>& _Arg) noexcept
{    // forward an lvalue as either an lvalue or an rvalue
    return (static_cast<_Ty&&>(_Arg));
}

CData* Creator(T&& t)
{
    return new CData(std::forward<T>(t));
}
```

1. 如果T为`std::string&`,那么`std::forward(t)` 返回值为`std::string&&&`, 折叠为`std::string&`，左值引用特性不变。
2. 如果T为`std::string&&`,那么`std::forward(t)` 返回值为`std::string&&&&`, 折叠为`std::string&&`，右值引用特性不变。
3. 如果调用者为`std::string`,调用将会转换成为`std::string&`，为类型1.



## 3. 完美转发的实际使用

完美转发在C++标准库中使用特别多，下面举一个例子：

### 3.1 make_shared

这个用来创建一个对象的智能指针，并且可以通过使用不同的参数来构造所创建对象的指针，这个函数实现如下：

```cpp
template<class _Ty,
    class... _Types>
_NODISCARD inline shared_ptr<_Ty> make_shared(_Types&&... _Args)
{    // make a shared_ptr
    const auto _Rx = new _Ref_count_obj<_Ty>(_STD forward<_Types>(_Args)...);  //参数完美转发

    shared_ptr<_Ty> _Ret;
    _Ret._Set_ptr_rep_and_enable_shared(_Rx->_Getptr(), _Rx);
    return (_Ret);
}
```

我们将模板参数全部完美转发给构造函数，类似的还有个`tuple`这些标准库。



## 4. 总结

完美转发使用两步来完成任务：

1. 在模板中使用&&接收参数。
2. 使用`std::forward()`转发给被调函数.

这样左值作为仍旧作为左值传递，右值仍旧作为右值传递！

> *原文链接:* *https://blog.csdn.net/xiangbaohui/article/details/103673177*



-- END --