## 1, 定义

**lambda 表达式就是一个函数（匿名函数**），也就**是一个没有函数名的函数**。为什么不需要函数名呢，因为**我们直接（一次性的）用它**，嵌入式用的它，不需要其他地方调用它。


lambda 表达式也叫**闭包**。闭就是封闭的意思（封闭就是其他地方都不调用它），包就是函数。


**lambda 表达式其实就是一个函数对象**，他内部创建了一个重载()操作符的类。


lambda 表达式的简单语法如下：**[capture] (parameters) -> return value { body }**, 只有 **[capture] 捕获列表和 { body } 函数体是必选的**，其他可选。



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