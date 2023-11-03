## 深入了解C++11中promise和future的使用

### 原理

C++11中promise和future机制是用于并发编程的一种解决方案，用于在不同线程完成数据传递（异步操作）。

传统方式通过回调函数处理异步返回的结果，导致代码逻辑分散且难以维护。

Promise和Future是一种提供访问异步操作结果的机制，可以在线程之间传递数据和异常消息。

应用场景：顾客在一家奶茶店点了单，服务员给顾客一个单号，当奶茶做好后，服务员更新排号的状态，顾客可以去做自己的事情了，顾客可以通过查询排号来得知奶茶是否做好，当查到奶茶做好了就可以回来取奶茶了。

```cpp
#include <IOStream>
#include <future>
#include <thread>

using namespace std;
using namespace std::chrono;

void WaitForMilkTea(future<int>& future)
{
  /*其中获取future结果有三种方式
   1、auto value = future.get()    get()方法会阻塞等待异步操作结束并返回结果
   2、std::future_status  方式判断状态 有deferred、timeout、ready三种状态
   3、可以
  */

  //future_status方法
  #if 0
  std::future_status status;
  do {
  status = future.wait_for(std::chrono::milliseconds(500));
  if (status == std::future_status::deferred) {
  std::cout << "deferred!!!" << std::endl; //异步操作还没开始
  } else if (status == std::future_status::timeout) {
  std::cout << "timeout!!!" << std::endl; //异步操作超时
  } else if (status == std::future_status::ready) {
  std::cout << "ready!!!" << std::endl; //异步操作已经完成
  }
  } while (status != std::future_status::ready);

  //通过判断future_status状态为ready时才通过get()获取值
  auto notice = future.get();
  std::cout << "WaitForMilkTea receive value:" << notice << std::endl;
  #endif

  //get()方法
  #if 0
  auto notice = future.get();   //get阻塞等待直到异步处理结束返回值
  std::cout << "WaitForMilkTea receive value:" << notice << std::endl;
  #endif

  //wait()方法
  future.wait(); //和get()区别是wait只等待异步操作完成，没有返回值
  auto notice = future.get();
  std::cout << "WaitForMilkTea receive value:" << notice << std::endl;
}

void MakeTea(promise<int>& promise)
{
  //do something 这里先睡眠5s
  std::this_thread::sleep_for(std::chrono::seconds(5));
  promise.set_value(1);
  std::this_thread::sleep_for(std::chrono::seconds(10));
  std::cout << "MakeTea Thread quit!!!" << std::endl;
}

int main()
{
  promise<int> pNotice;
  //获取与promise相关联的future
  future<int> pFuture = pNotice.get_future();

  thread Customer(WaitForMilkTea, ref(pFuture));
  thread Worker(MakeTea, ref(pNotice));

  Customer.join();
  Worker.join();
}
```

其中future_status枚举如下：

名称值示意ready0就绪timeout1等待超时deferred2延迟执行(与std::async配合使用)

future_errc 枚举 : 为 future_error 类报告的所有错误提供符号名称。

名称值示意broken_promise0与其关联的 std::promise 生命周期提前结束future_already_retrieved1重复调用 get() 函数promise_already_satisfied2与其关联的 std::promise 重复 setno_state4无共享状态



### Promise和Future模型

流程如下：

1.线程1初始化一个promise和future对象，将promise对象传递给线程2，相当于线程2对线程1的一个承诺

2.future相当于一个承诺，用于获取未来线程2的值

3.线程2接受一个promise，需要将处理结果通过promise返回给线程1

4.线程1想要获取数据，此时线程2还未返回promise就阻塞等待处，直到线程2的数据可达



### promise相关函数

std::future负责访问， std::future是一个模板类，它提供了可供访问异步执行结果的一种方式。

std::promise负责存储， std::promise也是一个模板类，它提供了存储异步执行结果的值和异常的一种方式。

总结：std::future负责访问，std::promise负责存储，同时promise是future的管理者

**std::future**

名称作用operator=移动 future 对象，移动！share()返回一个可在多个线程中共享的 std::shared_future 对象get()获取值（类型由模板类型决定）valid()检查 future 是否处于被使用状态，也就是它被首次在首次调用 get() 或 share() 前。建议使用前加上valid()判断wait()阻塞等待调用它的线程到共享值成功返回wait_for()在规定时间内 阻塞等待调用它的线程到共享值成功返回wait_until()在指定时间节点内 阻塞等待调用它的线程到共享值成功返回

1、普通构造函数， 默认无参构造函数

2、带自定义内存分配器的构造函数，与默认构造函数类似，但是使用自定义分配器来分配共享状态。

3、拷贝构造函数和普通=赋值运算符默认禁止

4、移动构造函数

5、移动赋值运算符

**std::future仅在创建它的std::promise(或者std::async、std::packaged_task)有效时才有用，所以可以在使用前用valid()判断**
**std::future可供异步操作创建者用各种方式查询、等待、提取需要共享的值，也可以阻塞当前线程等待到异步线程提供值。**
**std::future一个实例只能与一个异步线程相关联，多个线程则需要使用std::shared_future。**

**std::promise**

成员函数：

名称作用operator=从另一个 std::promise 移动到当前对象。swap()交换移动两个 std::promise。get_future()获取与其管理的std::futureset_value()设置共享状态值，此后promise共享状态标识变为readyset_value_at_thread_exit()设置共享状态的值，但是不将共享状态的标志设置为 ready，当线程退出时该 promise 对象会自动设置为 readyset_exception()设置异常，此后promise的共享状态标识变为readyset_exception_at_thread_exit()设置异常，但是到该线程结束时才会发出通知

1、std::promise::get_future：返回一个与promise共享状态相关联的future对象

2、std::promise::set_value：设置共享状态的值，此后promise共享状态标识变为ready

3、std::promise::set_exception：为promise设置异常，此后promise的共享状态标识变为ready

4、std::promise::set_value_at_thread_exit：设置共享状态的值，但是不将共享状态的标志设置为 ready，当线程退出时该 promise 对象会自动设置为 ready（注意：该线程已设置promise的值，如果在线程结束之后有其他修改共享状态值的操作，会抛出future_error(promise_already_satisfied)异常）

5、std::promise::swap：交换 promise 的共享状态

**std::promise的set相关函数\*php\*和get_future()只能被调用一次，多次调用会抛出异常**
**std::promise作为使用者的异步线程中应当注意到共享变量的生命周期、是否被set问题。如果没被set而线程就结束了，future端就会抛出异常。**



### 多线程std::shared_future

std::future 有个非常明显的问题，就是只能和一个 std::promise 成对绑定使用，也就意味着仅限于两个线程之间使用。

那么多个线程是否可以呢，可以！就是 std::shared_future。

std::shared_future 也是一个模板类，它的功能定位、函数接口和 std::future 一致，不同的是它允许给多个线程去使用，让多个线程去同步、共享：

它的语法是：【语法】【伪代码】std::shared_future<Type> s_fu(pt.get_future());



```cpp
#include <iostream>
#include <future>
#include <thread>

using namespace std;
using namespace std::chrono;

void futureHandleEntry(std::shared_future<int>& future) 
{
  if (future.valid()) {
  future.wait();
  std::cout << "thread:[" << std::this_thread::get_id() << "] value=" << future.get() << std::endl;
  std::cout << "thread:[" << std::this_thread::get_id() << "] quit!!!" << std::endl;
  }
  else {
  std::cout << "future is invalid!!!" << std::endl;
  }
}

int main()
{
    std::promise<int> promise;
    //获取shared_future用于多线程访问异步操作结果
    std::shared_future<int> future = promise.get_future();

    std::thread t1(&futureHandleEntry, ref(future));
    std::thread t2(&futureHandleEntry, ref(future));
    std::thread t3(&futureHandleEntry, ref(future));

    std::cout << "main thread running!!!" << std::endl;
    //主线程sleep5s后去set_value
    std::this_thread::sleep_for(std::chrono::seconds(5));
    promise.set_value(10);

    t1.join();
    t2.join();
    t3.join();
}
```



## promise和future进阶

我们知道异常的场景：

**1、当重复调用promise的set_value会导致抛出异常**

```cpp
#include <iostream>
#include <thread>
#include <future>
#include <chrono>

using namespace std;

void threadEntry(std::future<int>& future)
{
try {
auto value = future.get();
std::cout << "value=" << value << std::endl;
}
catch (std::future_error& error) {
std::cerr << error.code() << "\n" << error.what() << std::endl;
}
}
int main()
{
std::promise<int> promise;
std::future<int> future = promise.get_future();

std::thread t1(threadEntry, ref(future));
/*主线程promise多次调用set_value会抛出future_error异常
*/
std::this_thread::sleep_for(std::chrono::seconds(2));
promise.set_value(1); 
promise.set_value(1);

t1.join();
}
```

在linux中运行结果如下： 会有Promise already satisfied的错误提示

**2、 当std::promise不设置值时线程就退出**

如果promise直到销毁时，都未设置过任何值，则promise会在析构时自动设置为std::future_error，这会造成std::future.get抛出std::future_error异常。

```cpp
#include <iostream> // std::cout, std::endl
#include <thread>   // std::thread
#include <future>   // std::promise, std::future
#include <chrono>   // seconds
using namespace std::chrono;

void read(std::future<int> future) {
    try {
        future.get();
    } catch(std::future_error &e) {
        std::cerr << e.code() << "\n" << e.what() << std::endl;
    }
}

int main() {
    std::thread thread;
    {
        // 如果promise不设置任何值
        // 则在promise析构时会自动设置为future_error
        // 这会造成future.get抛出该异常
        std::promise<int> promise;
        thread = std::thread(read, promise.get_future());
   }
    thread.join();

    return 0;
}
```

**3、通过std::promise让std::future抛出异常**

通过std::promise.set_exception函数可以设置自定义异常，该异常最终会被传递到std::future，并在其get函数中被抛出。

```cpp
#include <iostream>
 #include <future>
 #include <thread>
 #include <exception>  // std::make_exception_ptr
 #include <stdexcept>  // std::logic_error
 
void catch_error(std::future<void> &future) {
    try {
       future.get();
    } catch (std::logic_error &e) {
       std::cerr << "logic_error: " << e.what() << std::endl;
   }
}

int main() {
    std::promise<void> promise;
    std::future<void> future = promise.get_future();

    std::thread thread(catch_error, std::ref(future));
    // 自定义异常需要使用make_exception_ptr转换一下
    promise.set_exception(
       std::make_exception_ptr(std::logic_error("caught")));

      thread.join();
     return 0;
}
```

std::promise虽然支持自定义异常，但它并不直接接受异常对象：

```
// std::promise::set_exception函数原型
2void set_exception(std::exception_ptr p);
```

自定义异常可以通过位于头文件exception下的std::make_exception_ptr函数转化为std::exception_ptr

**std::promise**

通过上面的例子，我们看到std::promise<void>

是合法的。此时std::promise.set_value不接受任何参数，仅用于通知关联的std::future.get()解除阻塞。

**std::promise所在线程退出时**

std::async(异步运行)时，开发人员有时会对std::promise所在线程退出时间比较关注。std::promise支持定制线程退出时的行为：

**std::promise.set_value_at_thread_exit 线程退出时，std::future收到通过该函数设置的值**
**std::promise.set_exception_at_thread_exit 线程退出时，std::future则抛出该函数指定的异常。**



## 项目碰到的坑

```cpp
// run in main thread
std::unique_ptr<Image> DisplayBank::dumpCurrentFrame() {
  {
    std::lock_guard<std::mutex> lk(mutex_);
    need_dump_current_frame_.store(true);
    promise_ = std::make_shared<std::promise<std::unique_ptr<Image>>>();
  }

  std::future<std::unique_ptr<Image>> future = promise_->get_future();
  std::unique_ptr<Image> image = nullptr;
  std::future_status status = future.wait_for(std::chrono::milliseconds(500));
  if (status == std::future_status::ready) {	// 注意要加这个判断，不然万一promise没有set_value过，析构后调future.get可能会抛异常
    image = future.get();
  }
  return image;
}

// run in GLThread
void DisplayBank::dumpCurrentFrameIfNeeded(std::shared_ptr<MediaFrame> frame) {
  if (frame && need_dump_current_frame_.load() == true) {
    need_dump_current_frame_.store(false);
    if (!dump_processor_ || !dump_processor_.get()) {
      ProcessorParam processor_param = ProcessorParam::Create(FilterCache::kFilterTypeBase,
                                                              kRenderModeFull | kRenderModeFill,
                                                              RenderTarget::kFramebuffer);
      processor_param.force_flip = true;
      if (!context_->is_multi_thread) {
        dump_processor_ = std::make_shared<BaseProcessor>(context_, processor_param);
      } else {
        auto work_processor = std::make_shared<BaseProcessor>(context_, processor_param);
        dump_processor_ = std::make_shared<EglProcessor>(context_, work_processor, "SSPCameraDisplayDump");
      }
    }

    if (dump_processor_) {
      dump_processor_->process(frame, [&](std::shared_ptr<MediaFrame> ret_frame) {
        std::unique_ptr<Image> image;
        auto dump_frame = std::dynamic_pointer_cast<VideoFrame>(ret_frame);
        if (dump_frame->framebuffer) {
          image = dump_frame->framebuffer->getImage(0, 0, dump_frame->width, dump_frame->height);
        } else if (dump_frame->texture) {
          image = dump_frame->texture->getImage(0, 0, dump_frame->width, dump_frame->height);
        }
        std::lock_guard<std::mutex> lk(mutex_);
        promise_->set_value(std::move(image));
      });
    } else {
      SLOGE("DisplayBank::dumpCurrentFrameIfNeeded: dump_processor_ is nullptr");
    }
  }
}
```

如上代码看起来已经做好了保护，不会导致重复promise_->set_value。但由于dump图片比较耗时，在std::lock_guard<std::mutex> lk(mutex_);之前可能部分低端机花较长时间。而主线程如果连续点击dumpCurrentFrame()。第一次点击，改变了原子变量need_dump_current_frame_，进入dump_processor_->process逻辑。但可能在处理getImage，并未进入到加锁逻辑。这时第二次主线程点击了dumpCurrentFrame，能得到锁，再次改变了need_dump_current_frame。GL线程在这时才处理完getImage，然后加锁->set_value->退出。但由于need_dump_current_frame又已经是true了。第二次GL线程会再次进入，导致重复set，出现异常：
The state of the promise has already been set.

修复方式可以是加大锁的返回，但这样可能导致主线程ANR。

修复后代码参考如下：

```cpp
std::unique_ptr<Image> DisplayBank::dumpCurrentFrame() {
  if (need_dump_current_frame_.load()) {  // 防重复点击
    return nullptr;
  }
  {
    std::lock_guard<std::mutex> lk(mutex_);
    need_dump_current_frame_.store(true);
    promise_ = std::make_shared<std::promise<std::unique_ptr<Image>>>();
  }

  std::future<std::unique_ptr<Image>> future = promise_->get_future();
  std::unique_ptr<Image> image = nullptr;
  // 增加获取图片成功率
  std::future_status status = future.wait_for(std::chrono::milliseconds(2000));
  if (status == std::future_status::ready) {
    image = future.get();
  }
  return image;
}

void DisplayBank::dumpCurrentFrameIfNeeded(std::shared_ptr<MediaFrame> frame) {
  if (need_dump_current_frame_.load()) {
    if (nullptr == frame) {
      need_dump_current_frame_.store(false);
      try {
        promise_->set_value(nullptr); // 避免主线程等2s
      } catch (const std::future_error& e) {
        SLOGE("DisplayBank::dumpCurrentFrameIfNeeded, the frame is nullptr, catch future_error: %s", e.what());
      }
      return ;
    }
    if (!dump_processor_ || !dump_processor_.get()) {
      ProcessorParam processor_param = ProcessorParam::Create(FilterCache::kFilterTypeBase,
                                                              kRenderModeFull | kRenderModeFill,
                                                              RenderTarget::kFramebuffer);
      processor_param.force_flip = true;
      if (!context_->is_multi_thread) {
        dump_processor_ = std::make_shared<BaseProcessor>(context_, processor_param);
      } else {
        auto work_processor = std::make_shared<BaseProcessor>(context_, processor_param);
        dump_processor_ = std::make_shared<EglProcessor>(context_, work_processor, "SSPCameraDisplayDump");
      }
    }

    if (dump_processor_) {
      dump_processor_->process(frame, [&](std::shared_ptr<MediaFrame> ret_frame) {
        std::unique_ptr<Image> image;
        auto dump_frame = std::dynamic_pointer_cast<VideoFrame>(ret_frame);
        if (dump_frame->framebuffer) {
          image = dump_frame->framebuffer->getImage(0, 0, dump_frame->width, dump_frame->height);
        } else if (dump_frame->texture) {
          image = dump_frame->texture->getImage(0, 0, dump_frame->width, dump_frame->height);
        }
        std::lock_guard<std::mutex> lk(mutex_);
        try {
          promise_->set_value(std::move(image));
        } catch (const std::future_error& e) {
          SLOGE("DisplayBank::dumpCurrentFrameIfNeeded catch future_error: %s", e.what());
        }
        need_dump_current_frame_.store(false);
      });
    } else {
      need_dump_current_frame_.store(false);
      try {
        promise_->set_value(nullptr); // 避免主线程等2s
      } catch (const std::future_error& e) {
        SLOGE("DisplayBank::dumpCurrentFrameIfNeeded, the dump_processor_ is nullptr, catch future_error: %s", e.what());
      }
      SLOGE("DisplayBank::dumpCurrentFrameIfNeeded: dump_processor_ is nullptr");
    }
  }
}
```

这种也防止了重复点击。