## 1, 定义

**lambda 表达式就是一个函数（匿名函数**），也就**是一个没有函数名的函数**。为什么不需要函数名呢，因为**我们直接（一次性的）用它**，嵌入式用的它，不需要其他地方调用它。


lambda 表达式也叫**闭包**。闭就是封闭的意思（封闭就是其他地方都不调用它），包就是函数。


**lambda 表达式其实就是一个函数对象**，他内部创建了一个重载()操作符的类。

lambda 表达式的简单语法如下：**[capture] (parameters) -> return value { body }**, 只有 **[capture] 捕获列表和 { body } 函数体是必选的**，其他可选。

* 用于capture 指定可在 lambda 中访问的周围作用域中的变量。变量可以按值、按引用或使用this.
* 指定parameters将传递给 lambda 的参数。
* return value指定 lambda 将返回的值的类型。如果未指定，编译器将尝试推导它。
* 指定 body调用 lambda 时将执行的代码。



## 2, 最简单的一个 lambda 表达式（调用）

```c++
int main()
{
    [] {}();//三部分，[] : 代表lambda表达式的开始;{} : 代表函数体，函数体里面什么都没有;() : 代表函数调用.
}

```

等价于:

```c++
void f()
{

}
int main()
{
    f();
}

```

像其他函数一样，我们需要一个参数列表：()。所以：
[] {}(); 也可以写为：

```c++
[](){}(); 
```

**第二个位置加了一个()代表函数参数。如果什么参数都没有，就可以省略 ()。**



## 3, 再加点打印的东西：

```c++
#include <iostream>
using namespace std;
int main()
{
     [] { cout << "Hello, World!"<<endl; }();
}

```

也可以这样：

```c++
#include <iostream>
using namespace std;
int main()
{
    auto lam = [] { cout << "Hello, World!"<<endl; };
    lam();
}

```



## 4, 返回值

-> int ：代表此匿名函数返回 int**。大多数情况**下lambda表达式的返回值可由编译器猜测得出，因此不需要我们指定返回值类型。

```c++
int main()
{
    auto lam =[]() -> int { cout << "Hello, World!"; return 88; };
    //auto lam =[]() { cout << "Hello, World!"; return 88; };//自动推导返回值
    auto ret = lam();
    cout<<ret<<endl;//输出88
    auto lam2 =[]() -> string { cout << "Hello, World!"; return "test"; };
    auto ret1 = lam2();
    cout<<ret1<<endl;//输出test
}
```



## 5, 捕捉变量

变量捕获才是成就 lambda 卓越的秘方。

**[] 不捕获任何变量**, **这种情况下lambda表达式内部不能访问外部的变量。**

**[&]** **以引用方式捕获所有变量**

**[=]** **用值的方式捕获所有变量**（可能被编译器优化为const &)


[=, &foo] 以引用捕获变量foo, 但其余变量都靠值捕获
[&, foo] 以值捕获foo, 但其余变量都靠引用捕获
[bar] 以值方式捕获bar; 不捕获其它变量
[this] 捕获所在类的this指针 （Qt中使用很多，如此lambda可以通过this访问界面控件的数据）

```c++
int a=1,b=2,c=3;
auto lam2 = [&,a](){//b，c以引用捕获，a以值捕获。
  b=5;c=6;//a =1; a不能赋值
  cout << a<<b<<c<<endl;//输出 1 5 6
};
lam2();
```



## 6, 配合STL使用

毋庸质疑，lambda最大的一个优势是在使用STL中的算法 (algorithms) 库时：

```c++
vector<string> address{"111","222",",333",".org","wwwtest.org"};
for_each(address.begin(),address.end(),[](const string& str){cout<<str<<endl;});
```

如此一行代码就可以循环打印容器的数据。
再看一个例子，以前数组排序代码(第二代排序，第一代是自己写)是这样的：

```c++
int arr[] = {6,4,3,2,1,5};
bool compare(int& a,int& b)//谓词函数
{
  return a>b;
}
std::sort(arr, arr+6, compare);
```

现在：

```c++
std::sort(arr, arr+6, [](const int& a,const int& b){return a>b;});//降序排序
//std::sort(arr, arr+6, [](const auto& a,const auto& b){return a>b;}); //C++14支持基于类型推断的泛型lambda表达式。
std::for_each(begin(arr),end(arr),[](const int& e){cout<<"After:"<<e<<endl;});//6,5,4,3,2,1
```

lambda表达式取代了谓词函数，一行代码搞定排序，再一行代码搞定打印数组所有数据。



## 7, 打印类型

```c++
auto lam = [](){cout<<"hi"<<endl;};
cout<<typeid(lam).name()<<endl;//输出 class <lambda_6c3f764f39072358acd689d114b4c204>
int i1;
cout<<"typeid:"<<typeid(i1).name()<<endl;//输出 i
cout<<"typeid:"<<typeid(float).name()<<endl;//输出 f
```



## 8, 举例

创建lambda函数的一个原因是有些人创建了一个希望接受(lambda函数)参数的函数。

**lambda 的引入给我们带来了一种全新的编程体验，它可以让我们把 “function” 当做是 “data” 一样传递**，并且使我们从繁琐的语法中解放出来，更加关注于 “算法” 本身。


**新的 std::function 是传递lambda函数的最好的方式**，不管是传递参数还是返回值。


以下代码将lambda表达式作为函数参数传递。程序的作用很简单，是从一个地址簿中查找满足条件的地址（匹配字符串或长度等规则）。( 参考: 「C++11」Lambda 表达式)

```c++
#include <iostream>
#include <string>
#include <vector>
#include <functional>
#include <algorithm>
using namespace std;

class AddressBook
{
public:
    //提供一个通用的查找方法，以供查询（匹配的地址），这个方法接受一个查找规则的函数作为参数
    std::vector<string> findMatchingAddresses (std::function<bool (const string&)> func)
    {
        std::vector<string> results;
        for ( auto it = _addresses.begin(), end = _addresses.end(); it != end; ++it ){
            // 调用传递到findMatchingAddresses的函数并检测是否匹配规则
            if ( func( *it ) ){
                results.push_back( *it );
            }
        }
        return results;
    }
    void SetAddress(const std::vector<std::string> &address)
    {
        _addresses = address;
    }
private:
    std::vector<string> _addresses;
};

//声明一个全局的AddressBook对象
AddressBook global_address_book;
//查找匹配名字的地址
vector<string> findAddressesFromName (const string &name)
{
    return global_address_book.findMatchingAddresses(
                [&] (const string& addr) { return addr.find(name) != string::npos; }
    );
}
//查找匹配长度的地址(>min_len)
vector<string> findAddressesLen (const size_t &min_len)
{
    return global_address_book.findMatchingAddresses( [&] (const string& addr) { return addr.length() >= min_len; } );
}

int main()
{
    //初始化AddressBook对象的成员 _addresses
    vector<string> address{"China chengdu","China hunan","TaiWan taibei","American alasijia","Japan dongjing"};
    global_address_book.SetAddress(address);
    //查找包含 China 的地址
    auto ret = findAddressesFromName("China");
    for_each(ret.begin(),ret.end(),[](string &i){cout<<i<<" ";});cout<<endl;//输出: China chengdu 和 China hunan
    //查找长度大于 15 的地址
    auto ret2 = findAddressesLen(15);
    for_each(ret2.begin(),ret2.end(),[](string &i){cout<<i<<" ";});cout<<endl;//输出: American alasijia

    return 0;
}
```

完。

版权声明：本文为CSDN博主「宇宙379」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/a379039233/article/details/83714770



Lambdas 提供了一种在 C++ 中编写类函数对象的灵活简洁的方法，并广泛用于现代 C++ 编程中。

以下是在 C++ 中使用 lambda 的几种不同方式：

1. 函数回调
2. 捕获默认值
3. 按价值捕获
4. 通过引用捕获
5. 可变的 Lambda

### {1} 函数回调

函数回调是作为参数传递给另一个函数的函数，稍后由接收函数调用。您可以将 lambda 作为函数参数传递，它会在特定事件发生时执行。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
  std::vector<int> numbers = {1, 2, 3, 4, 5};

  // Lambda expression to find the sum of two numbers
  auto sum = [](int a, int b) { return a + b; };

  int result = std::accumulate(numbers.begin(), numbers.end(), 0, sum);
  std::cout << "Sum of elements in the vector: " << result << std::endl;

  return 0;
}
```

在此示例中，sum变量是一个 Lambda 表达式，它接受两个参数a并b返回它们的总和。该std::accumulate函数采用数字向量、结果的初始值和求和函数（Lambda 表达式）。该函数计算向量中所有元素的总和并返回结果，该结果打印在屏幕上。

另一个例子：

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main()
{
    std::vector<int> vec = { 1, 2, 3, 4, 5 };
    int sum = 0;
    std::for_each(vec.begin(), vec.end(), [&sum](int x) { sum += x; });
    std::cout << "The sum is: " << sum << std::endl;
    return 0;
}
```

在这种情况下，lambda 表达式[&sum](int x) { sum += x; }作为要应用于每个元素的函数传递。sumlambda通过引用捕获变量&，以便可以在 lambda 主体内对其进行修改。

两个示例都获得了相同的结果，但第二个示例使用了std::for_each算法和 lambda 表达式，这是 C++ 中更现代和简洁的技术。

### {2} 捕获默认值

当声明 lambda 表达式时没有任何显式捕获，默认行为是通过引用捕获周围范围内的变量。这称为捕获默认值。

例子：

```cpp
#include <iostream>

int main() {
  int x = 42;
  auto f = [ ]() { std::cout << x << std::endl; };
  f();
  return 0;
}

#include <iostream>

int main()
{
    auto square = [](int x) { return x * x; };
    std::cout << "The square of 5 is: " << square(5) << std::endl;
    return 0;
}
```

在第二个示例中，lambda 表达式被定义并存储在名为 的变量中square。此 lambda 表达式接受一个int参数x并返回 的值x * x，即参数的平方。

在main函数中，这个 lambda 表达式被用作函数对象。它通过传递参数来调用5，结果使用cout流显示。

### {3} 按价值获取

这是 lambda 表达式的最简单形式，您可以在其中按值将变量传递给函数。当一个变量被值捕获时，它的当前值存储在闭包中，并且当变量在周围范围内发生变化时不会更新。这是通过在方括号中包含变量来完成的[ ]

例子

```cpp
#include <iostream>

int main() {
  int x = 42;
  auto f = [x]() { std::cout << x << std::endl; };
  f();
  return 0;
}

#include <iostream>

int main() {
    int x = 42;
    auto f = [x](int y) { std::cout << x+y << std::endl;};
    f(1);
    return 0;
}
```

### {4} 通过引用捕获

&您可以使用符号通过引用将变量传递给 lambda 表达式。当通过引用捕获变量时，其当前值存储在闭包中，并在变量在周围范围内发生变化时更新。这是通过&在方括号中的变量前面包含寻址运算符来完成的[ ]。

例子

```cpp
#include <iostream>

int main() {
  int x = 42;
  auto f = [&x]() { std::cout << x << std::endl; };
  f();
  return 0;
}
#include <iostream>

int main() {
  int x = 10;

  auto add_one = [&x]() { ++x; };
  add_one();
  std::cout << x << "\n";

  return 0;
}
#include <iostream>

int main() {
  int x = 42;
  auto f = [&x]() { std::cout << x << std::endl; };
  f();
  return 0;
}
```

在最后一个例子中，变量x是通过引用捕获的，lambdaadd可以修改它的值。

### {5} 可变的 Lambda

默认情况下，lambda 表达式捕获的变量是常量，不能在 lambda 表达式主体内修改。如果要修改 lambda 表达式中捕获的变量，可以使 lambda 表达式可变。可变 lambda 允许修改捕获的变量。这是通过mutable在方括号中包含关键字来完成的[ ]。

例子

```cpp
#include <iostream>

int main() {
  int x = 42;
  auto f = [x]() mutable { std::cout << ++x << std::endl; };
  f();
  return 0;
}
```

### {结论}

lambda 表达式类似于常规函数，但有一些关键差异。例如，lambda 表达式的类型没有明确指定，但可以由编译器推断。此外，lambda 表达式可以从周围范围捕获变量，这使得它们对于创建闭包和使用 C++ 中的函数式编程概念非常有用。

与传统函数相比，Lambda 具有一些性能优势

1. **内联函数**：Lambdas 由编译器自动内联，这意味着它们的代码直接插入到调用函数中。这可以减少函数调用的开销并提高性能。
2. **避免命名函数的开销：** Lambda 没有名称，因此不必声明它们并存储在符号表中，这样可以减少开销并提高性能。
3. **改进的缓存局部性**：Lambda 可以在同一函数中定义和使用，这意味着 lambda 使用的代码和数据将与调用代码存储在同一缓存行中。这可以改善缓存局部性并降低缓存未命中的成本。
4. **减少代码大小**：Lambda 通常比命名函数更小，并且不需要外部函数调用，这可以减少编译代码的大小并提高性能。
5. **增加灵活性**：Lambda 可用于将函数作为参数传递给其他函数，这在如何重用和组织代码方面提供了更大的灵活性。这可以通过减少对重复代码的需求来提高性能。
6. **提高可读性**：Lambda 可以通过以紧凑和独立的方式封装复杂的逻辑来提高代码的可读性。这可以通过使代码更容易理解和维护来提高性能。

总之，lambda 可以通过减少开销、改进缓存局部性、减少代码大小、增加灵活性和提高可读性来提供优于传统函数的性能。