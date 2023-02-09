当我们使用了new关键字去创建一个对象时，你知道背后做了哪些事情吗？

```cpp
A* a = new A;
```

实际上这样简单的一行语句， 背后做了以下三件事情：

1. 分配内存,如果类A重载了operator new，那么将调用A::operator new(size_t )来完成，如果没有重载，就调用::operator new(size_t )，即全局new操作符来完成。
2. 调用构造函数生成类对象；
3. 返回相应指针。

下面我们通过一个例子来验证这个过程：

```cpp
#include <iostream>
#include <string>
#include <malloc.h>
using namespace std;

//student class
class Stu
{
public:
    Stu(string name, int age)
    {
        cout << "call Stu class constructor" << endl; 
        name_ = name;
        age_ = age;
    };
public:
    void print() const
    {
        cout << "name = " << name_ << std::endl;
        cout<< "age = " << age_ << std::endl;
    };
    void* operator new(size_t size)
    {
        std::cout << "call operator new" << std::endl;
        return malloc(size);
    }
private:
    string name_;
    int age_;
};
int main()
{
    Stu* stu1 = new Stu("a", 10);
}
```

在上述代码中，我们重载了Stu类的operator new操作符，用来验证上述对于new关键字的描述。

上述代码的执行结果如下所示：

```cpp
call operator new
call Stu class constructor
```

可以看到重载的operator new被调用，类Stu的构造函数也被调用，验证了上述的描述。

要注意到的是new是一个关键字，和sizeof一样，我们不能修改其具体功能。

# operator new

从new的调用过程中，我们知道会调用operator new操作符

那么operator new又是什么呢？

C++支持运算符的重载，支持对一些运算符自定义其行为：

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/f2b3f9f688f74f06991cdb80b360f9e8~noop.image?_iz=58558&from=article.pc_detail&x-expires=1676283340&x-signature=T9tKLdhZ4HFE1hlEK89WrG19IE8%3D)



operator new

operator new是一个操作符，和+ -操作符一样，作用是分配空间。我们可以重写它们，修改分配空间的方式。

operator new返回值必须是void*。第一个参数必须是size_t

```cpp
void* operator new (std::size_t size) throw (std::bad_alloc);  
void* operator new (std::size_t size, const std::nothrow_t& nothrow_constant) throw();  
```

在下面的例子中，我们使用重载了三个operator new方法， 并分别调用。

```cpp
#include <iostream>
#include <string>
#include <malloc.h>
using namespace std;

//student class
class Stu
{
public:
    Stu(string name, int age)
    {
        cout << "call Stu class constructor" << endl; 
        name_ = name;
        age_ = age;
    };
    ~Stu()
    {
        cout << "call destructor"  << endl;
    }
public:
    void print() const
    {
        cout << "name = " << name_ << std::endl;
        cout<< "age = " << age_ << std::endl;
    };
    void* operator new(size_t size)
    {
        std::cout << "call operator new" << std::endl;
        return malloc(size);
    }
    void* operator new(size_t size, int num)
    {
        std::cout << "call operator new with int" << std::endl;
        return malloc(size);
    } 
    void* operator new(size_t size, char c)
    {
        std::cout << "call operator new with char" << std::endl;
        return malloc(size);
    }      
private:
    string name_;
    int age_;
};
int main()
{
    Stu* stu1 = new Stu("a", 10);
    Stu* stu2 = new(1) Stu("a", 10);
    Stu* stu3 = new('c') Stu("a", 10);
    delete stu1;
    delete stu2;
    delete stu3;
}
```

执行结果如下：

```shell
call operator new
call Stu class constructor
call operator new with int
call Stu class constructor
call operator new with char
call Stu class constructor
call destructor
call destructor
call destructor
```

可以看到重载的三个operator new被成功调用。

# placement new

placement new是operator new的一种重载形式，其作用是可以**在指定的内存地址创建对象**。

placement new返回值必须是void*。第一个参数必须是size_t， 第二个参数是void*

```cpp
void* operator new (std::size_t size, void* ptr) throw();  
```

下面的是一个关于placement new的调用例子:

```cpp
#include <iostream>
#include <string>
#include <malloc.h>
using namespace std;

//student class
class Stu
{
public:
    Stu(string name, int age)
    {
        name_ = name;
        age_ = age;
    };
    ~Stu()
    {
        cout << "call destructor"  << endl;
    }
public:
    void print() const
    {
        cout << "name = " << name_ << std::endl;
        cout<< "age = " << age_ << std::endl;
    };
    void* operator new(size_t size, void* p)
    {
        std::cout << "placement new" << std::endl;
        return p;
    };    
private:
    string name_;
    int age_;
};
int main()
{
    char* buff = (char*)malloc(sizeof(Stu));
    Stu *stu = new (buff) Stu("stu1", 10);
    stu->print();
    stu->~Stu();
    free(buff);
}
```

执行结果如下：

```shell
placement new
name = stu1
age = 10
call destructor
```

由于placement new可以在一个指定的位置创建对象，因此在STL中有很广泛的运用， 例子vector容器初始化的时候，会使用allocator申请一定的内存，当使用push_back放入对象时， 就可以使用placement new在申请的位置创建对象。

这里以MyTinySTL中创建对象的函数为例，construct.hopen in new window， 可以看出construct函数就是使用了全局的placement new方法在指定地址创建对象。

```cpp
template <class Ty, class... Args>
void construct(Ty* ptr, Args&&... args)
{
  ::new ((void*)ptr) Ty(mystl::forward<Args>(args)...);
}
```

需要注意的是， placement new只是在指定的内存地址上构建对象，在对象使用完毕之后，需要**手动调用析构函数**做资源的析构。

```cpp
char* buff = (char*)malloc(4096);
Stu *stu = new (buff) Stu("stu1", 10);
stu->print();
stu->~Stu();
free(buff);
```

为什么需要这样呢？

我们知道，使用delete关键字去释放一个对象时，首先会调用析构函数，然后再调用operator delete释放内存。

但是由于内存的申请并不是通过new对象而申请的，而是通过malloc申请到的一块内存空间，placement new只有"借用"了该内存空间去构建了对象， 因此是不能使用delete关键字去直接释放该对象的。由于不能直接调用delete stu，这就会导致对象的析构函数不会被自动调用，因此我们需要手动调用对象的析构函数做对象的析构stu->~Stu()，最后使用与malloc配套的free释放申请到的内存空间。

此外，我们讨论另外一个问题， placement new可以在栈上构建对象吗？

答案是肯定的。

```cpp
#include <iostream>
#include <string>
#include <malloc.h>
using namespace std;

//student class
class Stu
{
public:
    Stu(string name, int age)
    {
        name_ = name;
        age_ = age;
    };
    ~Stu()
    {
        cout << "call destructor"  << endl;
    }
public:
    void print() const
    {
        cout << "name = " << name_ << std::endl;
        cout<< "age = " << age_ << std::endl;
    };
    void* operator new(size_t size, void* p)
    {
        std::cout << "placement new" << std::endl;
        return p;
    };    
private:
    string name_;
    int age_;
};
int main()
{
    char buff[4096];
    Stu *stu = new (buff) Stu("stu1", 10);
    stu->print();
    stu->~Stu();
}
```

与在堆上调用placement new一样， 同样需要手动调用析构函数做资源的释放。但是由于内存是在栈上的， 因此不需要手动释放。

# 结论

对于new， operator new 和 placement new三者的区别， 我们总结如下：

**new**：

new是一个关键字，不能被重载。

new 操作符的执行过程如下：

1. 调用operator new分配内存 ；
2. 调用构造函数生成类对象；
3. 返回相应指针。

**operator new**：

operator new就像operator + 一样，是**可以重载**的。如果类中没有重载operator new，那么调用的就是**全局的::operator new**来完成堆的分配。同理，operator new[]、operator delete、operator delete[]也是可以重载的。

**placement new**：

**placement new和operator new并没有本质区别**。它们都是operator new操作符的重载，只是参数不相同。

placement并不分配内存，只是返回指向已经分配好的某段内存的一个指针。因此不能使用delete关键字删除它，需要手动调用对象的析构函数。

如果你想在**已经分配的内存**中创建一个对象，使用new时行不通的。也就是说placement new允许你在一个已经分配好的内存中（栈或者堆中）构造一个新的对象。原型中void* p实际上就是指向一个已经分配好的内存缓冲区的的首地址。