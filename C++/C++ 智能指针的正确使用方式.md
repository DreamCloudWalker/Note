来自https://www.cyhone.com/articles/right-way-to-use-cpp-smart-pointer/



C++11 中推出了三种智能指针，unique_ptr、shared_ptr 和 weak_ptr，同时也将 auto_ptr 置为废弃 (deprecated)。

但是在实际的使用过程中，很多人都会有这样的问题：

1. 不知道三种智能指针的具体使用场景
2. 无脑只使用 shared_ptr
3. 认为应该禁用 raw pointer(裸指针，即 Widget * 这种形式)，全部使用智能指针

本文将从这几方面讲解智能指针：

1. 智能指针的应用场景分析
2. 智能指针的性能分析: 为什么 shared_ptr 性能比 unique_ptr 差
3. 指针作为函数参数时应该传，传值、传引用，还是裸指针？



# 对象所有权

首先需要理清楚的概念就是对象所有权的概念。所有权在 rust 语言中非常严格，写 rust 的时候必须要清楚自己创建的每个对象的所有权。

但是 C++ 比较自由，似乎我们不需要明白对象的所有权，写的代码也能正常运行。但是明白了对象所有权，我们才可以正确管理好对象生命周期和内存问题。

C++ 引入了智能指针，也是为了更好的描述对象所有权，简化内存管理，从而大大减少我们 C++ 内存管理方面的犯错机会。



# unique_ptr：专属所有权

**我们大多数场景下用到的应该都是 unique_ptr**。
unique_ptr 代表的是专属所有权，即由 unique_ptr 管理的内存，只能被一个对象持有。
所以，**unique_ptr 不支持复制和赋值**，如下：

```cpp
auto w = std::make_unique<Widget>();
auto w2 = w; // 编译错误
```

如果想要把 w 复制给 w2, 是不可以的。因为复制从语义上来说，两个对象将共享同一块内存。

因此，**unique_ptr 只支持移动**, 即如下：

```cpp
auto w = std::make_unique<Widget>();
auto w2 = std::move(w); // w2 获得内存所有权，w 此时等于 nullptr
```

unique_ptr 代表的是专属所有权，如果想要把一个 unique_ptr 的内存交给另外一个 unique_ptr 对象管理。**只能使用 std::move 转移当前对象的所有权**。转移之后，当前对象不再持有此内存，新的对象将获得专属所有权。

如上代码中，将 w 对象的所有权转移给 w2 后，w 此时等于 nullptr，而 w2 获得了专属所有权。



## 性能

因为 C++ 的 zero cost abstraction 的特点，unique_ptr 在默认情况下和裸指针的大小是一样的。
所以 **内存上没有任何的额外消耗，性能是最优的**



## 使用场景 1：忘记 delete

unique_ptr 一个最简单的使用场景是用于类属性。代码如下：

```cpp
class Box{
public:
    Box() : w(new Widget())
    {}

    ~Box()
    {
        // 忘记 delete w
    }
private:
    Widget* w;
};
```

如果因为一些原因，w 必须建立在堆上。如果用裸指针管理 w，那么需要在析构函数中 `delete w`;
这种写法虽然没什么问题，但是容易漏写 delete 语句，造成内存泄漏。

如果按照 unique_ptr 的写法，不用在析构函数手动 delete 属性，当对象析构时，属性 `w` 将会自动释放内存。



## 使用场景 2：异常安全

假如我们在一段代码中，需要创建一个对象，处理一些事情后返回，返回之前将对象销毁，如下所示：

```c++
void process()
{
    Widget* w = new Widget();
    w->do_something(); // 可能会发生异常
    delete w;
}
```

在正常流程下，我们会在函数末尾 delete 创建的对象 w，正常调用析构函数，释放内存。

但是如果 w->do_something() 发生了异常，那么 `delete w` 将不会被执行。此时就会发生 **内存泄漏**。
我们当然可以使用 try…catch 捕捉异常，在 catch 里面执行 delete，但是这样代码上并不美观，也容易漏写。

如果我们用 std::unique_ptr，那么这个问题就迎刃而解了。无论代码怎么抛异常，在 unique_ptr 离开函数作用域的时候，内存就将会自动释放。



# shared_ptr：共享所有权

在使用 shared_ptr 之前应该考虑，是否真的需要使用 shared_ptr, 而非 unique_ptr。

shared_ptr 代表的是共享所有权，即多个 shared_ptr 可以共享同一块内存。
因此，从语义上来看，**shared_ptr 是支持复制的**。如下：

```cpp
auto w = std::make_shared<Widget>();
{
    auto w2 = w;
    cout << w.use_count() << endl;  // 2
}
cout << w.use_count() << endl;  // 1
```

shared_ptr 内部是利用引用计数来实现内存的自动管理，每当复制一个 shared_ptr，引用计数会 + 1。当一个 shared_ptr 离开作用域时，引用计数会 - 1。当引用计数为 0 的时候，则 delete 内存。

同时，**shared_ptr 也支持移动**。从语义上来看，移动指的是所有权的传递。如下：

```none
auto w = std::make_shared<Widget>();
auto w2 = std::move(w); // 此时 w 等于 nullptr，w2.use_count() 等于 1
```

我们将 w 对象 move 给 w2，意味着 w 放弃了对内存的所有权和管理，此时 w 对象等于 nullptr。
而 w2 获得了对象所有权，但因为此时 w 已不再持有对象，因此 w2 的引用计数为 1。



## 性能

1. **内存占用高**
   shared_ptr 的内存占用是裸指针的两倍。因为除了要管理一个裸指针外，还要维护一个引用计数。
   因此相比于 unique_ptr, shared_ptr 的内存占用更高
2. **原子操作性能低**
   考虑到线程安全问题，引用计数的增减必须是原子操作。而原子操作一般情况下都比非原子操作慢。
3. **使用移动优化性能**
   shared_ptr 在性能上固然是低于 unique_ptr。而通常情况，我们也可以尽量避免 shared_ptr 复制。
   如果，一个 shared_ptr 需要将所有权共享给另外一个新的 shared_ptr，而我们确定在之后的代码中都不再使用这个 shared_ptr，那么这是一个非常鲜明的移动语义。
   对于此种场景，我们尽量使用 std::move，将 shared_ptr 转移给新的对象。因为移动不用增加引用计数，性能比复制更好。



## 使用场景

1. shared_ptr 通常使用在共享权不明的场景。有可能多个对象同时管理同一个内存时。
2. 对象的延迟销毁。陈硕在《Linux 多线程服务器端编程》中提到，当一个对象的析构非常耗时，甚至影响到了关键线程的速度。可以使用 `BlockingQueue<std::shared_ptr<void>>` 将对象转移到另外一个线程中释放，从而解放关键线程。



## 为什么要用 shared_from_this?

我们往往会需要在类内部使用自身的 shared_ptr，例如：

```cpp
class Widget
{
public:
    void do_something(A& a)
    {
        a.widget = 该对象的 shared_ptr;
    }
}
```

我们需要把当前 shared_ptr 对象同时交由对象 a 进行管理。意味着，当前对象的生命周期的结束不能早于对象 a。因为对象 a 在析构之前还是有可能会使用到 `a.widget`。

如果我们直接 `a.widget = this;`， 那肯定不行， 因为这样并没有增加当前 shared_ptr 的引用计数。shared_ptr 还是有可能早于对象 a 释放。

如果我们使用 `a.widget = std::make_shared<Widget>(this);`，肯定也不行，因为这个新创建的 shared_ptr，跟当前对象的 shared_ptr 毫无关系。当前对象的 shared_ptr 生命周期结束后，依然会释放掉当前内存，那么之后 `a.widget` 依然是不合法的。

对于这种，需要在对象内部获取该对象自身的 shared_ptr, 那么该类必须继承 `std::enable_shared_from_this<T>`。代码如下:

```cpp
class Widget : public std::enable_shared_from_this<Widget>
{
public:
    void do_something(A& a)
    {
        a.widget = shared_from_this();
    }
}
```

这样才是合法的做法。



# weak_ptr

#### 声明

头文件：`<memory>` 
模版类：`template <class T> class weak_ptr` 
声明方式：`std::weak_ptr<type_id> statement`

weak_ptr 是为了解决 shared_ptr 双向引用的问题。即：

```cpp
class B;
struct A{
    shared_ptr<B> b;
};
struct B{
    shared_ptr<A> a;
};
auto pa = make_shared<A>();
auto pb = make_shared<B>();
pa->b = pb;
pb->a = pa;
```

pa 和 pb 存在着循环引用，根据 shared_ptr 引用计数的原理，pa 和 pb 都无法被正常的释放。
对于这种情况, 我们可以使用 weak_ptr：

```cpp
class B;
struct A{
    shared_ptr<B> b;
};
struct B{
    weak_ptr<A> a;
};
auto pa = make_shared<A>();
auto pb = make_shared<B>();
pa->b = pb;
pb->a = pa;
```

weak_ptr 不会增加引用计数，因此可以打破 shared_ptr 的循环引用。
通常做法是 parent 类持有 child 的 shared_ptr, child 持有指向 parent 的 weak_ptr。这样也更符合语义。



### 总结

shared_ptr和weak_ptr主要区别如下

1. shared_ptr对象能够初始化实际指向一个地址内容而weak_ptr对象没办法直接初始化一个具体地址，它的对象需要由shared_ptr去初始化
2. weak_ptr不会影响shared_ptr的引用计数，因为它是一个弱引用，只是一个临时引用指向shared_ptr。即使用shared_ptr对象初始化weak_ptr不会导致shared_ptr引用计数增加。依此特性可以解决shared_ptr的循环引用问题。
3. weak_ptr没有解引用*和获取指针->运算符,它只能通过lock成员函数去获取对应的shared_ptr智能指针对象，从而获取对应的地址和内容。

参考文档： 
http://www.cplusplus.com/reference/memory/weak_ptr/ 
https://www.boost.org/doc/libs/1_66_0/libs/smart_ptr/doc/html/smart_ptr.html#weak_ptr



# 选择哪种指针作为函数的参数

很多时候，函数的参数是个指针。这个时候就会面临选择困难症，这个参数应该怎么传，应该是 shared_ptr，还是 const shared_ptr&，还是直接 raw pointer 更合适。

1. **只在函数使用指针，但并不保存对象内容**
   假如我们只需要在函数中，用这个对象处理一些事情，但不打算涉及其生命周期的管理，也不打算通过函数传参延长 shared_ptr 的生命周期。
   对于这种情况，可以使用 raw pointer 或者 const shared_ptr&。
   即：

   ```cpp
   void func(Widget*);
   void func(const shared_ptr<Widget>&)
   ```

   实际上第一种裸指针的方式可能更好，从语义上更加清楚，函数也不用关心智能指针的类型。

2. **在函数中保存智能指针**
   假如我们需要在函数中把这个智能指针保存起来，这个时候建议直接传值。

   ```cpp
   void func(std::shared_ptr<Widget> ptr);
   ```

   这样的话，外部传过来值的时候，可以选择 move 或者赋值。函数内部直接把这个对象通过 move 的方式保存起来。
   这样性能更好，而且外部调用也有多种选择。



# 总结

对于智能指针的使用，实际上是对所有权和生命周期的思考，一旦想明白了这两点，那对智能指针的使用也就得心应手了。
同时理解了每种智能指针背后的性能消耗、使用场景，那智能指针也不再是黑盒子和洪水猛兽。



# 参考

- 《Effective Modern cpp》
- 《Linux 多线程服务器端编程》
- [GotW #91 Solution: Smart Pointer Parameters](https://herbsutter.com/2013/06/05/gotw-91-solution-smart-pointer-parameters/)
- [std::enable_shared_from_this 有什么意义？](https://www.zhihu.com/question/30957800)

*如果你在阅读过程中发现本文有错误或者存疑之处，请在下方评论区或者通过公众号进行留言，作者会尽快回复和解答。感谢您的支持与帮助。*