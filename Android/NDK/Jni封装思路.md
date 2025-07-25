### Jni封装要素

#### 1. 引用

JNI的引用分为三种Local References、Global References和Weak Global References，他们的关系可用简单理解为C++的堆、栈，引用。

* Local References
  大部分JNI方法返回的引用类型（例如**FindClass**）/ Jni函数内部创建的 `jobject `对象及其子类( `jclass `、 `jstring `、 `jarray  `、 `jobject `等) 对象都是local引用，local引用仅仅在此native方法中有效，当native方法返回时此local引用会被自动释放掉，**也可以**调用<font color="red">DeleteLocalRef</font>手动释放。<font color="red">local引用的个数是有限制的</font>，所以建议当不使用的时候就手动释放一下。一般情况下，我们应该依赖 JVM 去自动释放 JNI 局部引用；但下面两种情况必须手动调用 `DeleteLocalRef() `去释放：
  
  * (在循环体或回调函数中)创建大量 JNI 局部引用，即使它们并不会被同时使用，因为 JVM 需要足够的空间去跟踪所有的 JNI 引用，所以可能会造成内存溢出或者栈溢出;
  * 如果对一个大的 Java 对象创建了 JNI 局部引用，也必须在使用完后手动释放该引用，否则 GC 迟迟无法回收该 Java 对象也会引发内存泄漏.
  * 长时间泄漏不释放LocalRef可能导致崩溃：SIGABRT: Abort program: art/runtime/indirect_reference_table.cc:184] JNI ERROR (app bug): local reference table overflow (max=512)。参考https://stackoverflow.com/questions/26685491/jni-error-app-bug-local-reference-table-overflow-max-512。

* Global References
  Global引用在整个生命周期中都是有效的（直到手动释放它），同样它的<font color="red">引用个数也是有限的</font>，所以在不需要的时候需要手动释放一下。使用NewGlobalRef创建一个Global引用，使用DeleteGlobalRef删除一个Global引用。

* Weak Global References
  Weak Global如其名，所以在使用的时候需要判断一下这个应用对象是否还可用<font color="red">IsSameObject(o, nullptr)</font>，同样它的<font color="red">引用个数也是有限的</font>，所以在不需要的时候需要手动释放一下。使用NewWeakGlobalRef创建一个Weak Global引用，使用DeleteWeakGlobalRef删除一个Weak Global引用。

* 在 JNI 层执行 Java 代码常用到 `FindClass() `、 `GetMethodID() `、 `GetFieldID() `；但只有第一个函数返回的 `jclass `属于 JNI (局部)引用对象，<font color="red">而 `jmethodID `和 `jfieldID `并不是，它们是指向内部 Runtime 数据结构的指针</font>；实际上这些 ID 是用于缓存的静态对象：第一次查找会做一次字符串比较，但后面再次调用就能直接读取而变得很快；JVM 会保证这些 ID 是合法的，直到 `Class `被 unload；所以， `jmethodID `和 `jfieldID `是不需要手动释放的，当然也不能作为 JNI 全局引用。
  
  参考https://my.oschina.net/u/4419414/blog/4320396

* **其他非 JNI 引用**:
  
  除了上面提到的 ID，类似 `GetStringUTFChars() `和 `GetByteArrayElements() `/ `GetCharArrayElements() `等函数返回的也是 Raw Data 指针，而非 JNI 引用；
  
  在调用相对应的 `ReleaseXXX() `函数释放前，它们都是合法的；

* 同一个 `jobject `对象的不同引用可能拥有不同的值，比如同一 `jobject `对象每次调用 `NewGlobalRef() `可能返回不同的值；
  
  要检查两个引用是否指向同一个 `jobject `对象，必须调用 `IsSameObject() `，而不要使用 `== `去比较;

#### 2.  Jni多线程（JNIEnv封装）

JNIEnv是线程相关的数据结构，每一个Java线程存储一个对应的JNIEnv对象(`pthread_setspecific`)，JNIEnv是Java和C/C++交互的桥梁。Native方法中的JNIEnv参数是从调用线程中获取的，所以不同线程调用这个JNIEnv是不同的。

* JNIEnv可以跨函数，但不能跨线程，否则崩溃；解决方案：

```c++
// 假设这里是个异步线程执行的函数
JNIEnv *env = nullptr;
jint attachRet = ::javaVm->AttachCurrentThread(&env, nulltpr);   // 附加当前异步线程后，会得到一个全新的JNIEnv,是该子线程专用env
if (JNI_OK != attachRet) {
    return ;
}

// 用完后要Detach，不然会报错
::javaVm->DetachCurrentThread();
```

* jobject即不能跨函数也不能跨线程，否则崩溃；默认是局部引用，可升级为全局引用解决问题。

* JavaVM（一个进程只有一个）能跨线程和跨函数；

* 以下是WebRTC内部的一个实现，其实Android源码里面的实现也是一样的。我简单说明一下，当一个线程绑定了一个JNIEnv，我们可以通过GetEnv获取对应的JNIEnv，当一个线程没有绑定JNIEnv，我们可以通过AttachCurrentThread为当前线程绑定一个JNIEnv。那么当线程退出的时候我们如何释放这个JNIEnv呢？通过pthread_setspecific把这个JNIEnv存到线程中，当线程退出的时候通过pthread_getspecific取出，然后释放掉它就好了。

```c++
// Return a |JNIEnv*| usable on this thread or NULL if this thread is detached.
JNIEnv* GetEnv()
{
  void* env = nullptr;
  jint status = g_jvm->GetEnv(&env, JNI_VERSION_1_6);
  RTC_CHECK(((env != nullptr) && (status == JNI_OK)) ||
            ((env == nullptr) && (status == JNI_EDETACHED)))
      << "Unexpected GetEnv return: " << status << ":" << env;
  return reinterpret_cast<JNIEnv*>(env);
}

static void ThreadDestructor(void* prev_jni_ptr) {
  // This function only runs on threads where |g_jni_ptr| is non-NULL, meaning
  // we were responsible for originally attaching the thread, so are responsible
  // for detaching it now.  However, because some JVM implementations (notably
  // Oracle's http://goo.gl/eHApYT) also use the pthread_key_create mechanism,
  // the JVMs accounting info for this thread may already be wiped out by the
  // time this is called. Thus it may appear we are already detached even though
  // it was our responsibility to detach!  Oh well.
  if (!GetEnv())
    return;

  RTC_CHECK(GetEnv() == prev_jni_ptr)
      << "Detaching from another thread: " << prev_jni_ptr << ":" << GetEnv();
  jint status = g_jvm->DetachCurrentThread();
  RTC_CHECK(status == JNI_OK) << "Failed to detach thread: " << status;
  RTC_CHECK(!GetEnv()) << "Detaching was a successful no-op???";
}

static void CreateJNIPtrKey() {
  RTC_CHECK(!pthread_key_create(&g_jni_ptr, &ThreadDestructor))
      << "pthread_key_create";
}

// Return thread ID as a string.
static std::string GetThreadId()
{
    char buf[21];  // Big enough to hold a kuint64max plus terminating NULL.
    RTC_CHECK_LT(snprintf(buf, sizeof(buf), "%ld", static_cast<long>(syscall(__NR_gettid))), sizeof(buf))
        << "Thread id is bigger than uint64??";

    return std::string(buf);
}

// Return the current thread's name.
static std::string GetThreadName() {
    char name[17] = {0};
    if (prctl(PR_GET_NAME, name) != 0)
        return std::string("<noname>");
    return std::string(name);
}

JNIEnv* AttachCurrentThreadIfNeeded()
{
    JNIEnv* jni = GetEnv();
    if (jni)
        return jni;

    RTC_CHECK(!pthread_getspecific(g_jni_ptr)) << "TLS has a JNIEnv* but not attached?";

    std::string name(GetThreadName() + " - " + GetThreadId());
    JavaVMAttachArgs args;
    args.version = JNI_VERSION_1_6;
    args.name = &name[0];
    args.group = nullptr;
    // Deal with difference in signatures between Oracle's jni.h and Android's.
#ifdef _JAVASOFT_JNI_H_  // Oracle's jni.h violates the JNI spec!
    void* env = nullptr;
#else
    JNIEnv* env = nullptr;
#endif
    RTC_CHECK(!g_jvm->AttachCurrentThread(&env, &args)) << "Failed to attach thread";
    RTC_CHECK(env) << "AttachCurrentThread handed back NULL!";
    jni = reinterpret_cast<JNIEnv*>(env);
    RTC_CHECK(!pthread_setspecific(g_jni_ptr, jni)) << "pthread_setspecific";
    return jni;
}
```

WebRTC是在线程释放时做DetachCurrentThread，通过pthread_key_create传的销毁函数。

另外也可以参考谷歌Filament引擎的的实现，这个相对比较简单直观，直接在析构里DetachCurrentThread了：

https://github.com/google/filament/blob/main/filament/backend/include/private/backend/VirtualMachineEnv.h

外部使用JNIEnv时通过::get()::getEnvironment()来获取，由static thread_local jvm instance;保证析构函数的执行。

#### 3. Native异常捕获

理论上来说调用大部分JNI接口都需要在调用之后判断一下是否有错误，如果不检查是否存在错误，那么会在下一个调用的时候直接奔溃，不利于问题定位。一般来说这类接口都需要在调用之后判断一下：Get<Static>MethodID、Get<Static><TYPE>Field、Call<Static><TYPE>Method、NewGlobalRef、DeleteGlobalRef、NewStringUTF等。

```C++
if (env->ExceptionCheck()) {
   env->ExceptionDescribe();
   env->ExceptionClear();
}
```

如果需要Java层抛出异常需要封装，详情见笔记《JNI or NDK总结笔记》中的native异常捕获

可以参考WebRTC封装成宏定义如下：

```c++
#define CHECK_EXCEPTION(jni, ...)        \
  if (jni->ExceptionCheck()) {      \
     jni->ExceptionDescribe();      \
     jni->ExceptionClear();         \
     LOGE(__VA_ARGS__); }
```

#### 4. 方法调用

做一层封装，给外部统一调用。

* 构造函数
  
  构造函数的方法名是`<init>`

```c++
jclass cla = env->FindClass(env, "com/dj/Test");

// 构造函数一
jmethodID method = env->GetMethodID(cla, "<init>", "()V");
jobject cla = env->NewObject(cla, method);

// 构造函数2
method = env->GetMethodID(cla, "<init>", "(I)V");
cla = env->NewObject(cla, method, 100);

// 构造函数3
method = env->GetMethodID(cla, "<init>", "(II)V");
cla = env->NewObject(cla, method, 200, 300);

// 构造函数4
method = env->GetMethodID(cla, "<init>", "(III)V");
cla = env->NewObject(cla, method, 400, 500, 600);
```

* 静态方法

```c++
// 1.寻找类 Student
// jclass studentClass = env->FindClass("com/dengjian/Student"); // 第一种
jclass studentClass =  env->GetObjectClass(student); // 第二种

// 2.Student类里面的函数规则  签名
jmethodID showInfo = env->GetStaticMethodID(studentClass, "showInfo", "(Ljava/lang/String;)V");

// 3.调用静态showInfo
jstring  jstringValue = env->NewStringUTF("静态方法你好，我是C++");
env->CallStaticVoidMethod(studentClass, showInfo, jstringValue);
```

* 对象方法

```C++
// 1.寻找类 Student
// jclass studentClass = env->FindClass("com/dengjian/Student"); // 第一种
jclass studentClass =  env->GetObjectClass(student); // 第二种

// 2.Student类里面的函数规则  签名
jmethodID setName = env->GetMethodID(studentClass, "setName", "(Ljava/lang/String;)V");
jmethodID getName = env->GetMethodID(studentClass, "getName", "()Ljava/lang/String;");

// 3.调用 setName
jstring value = env->NewStringUTF("AAAA");
env->CallVoidMethod(student, setName, value);

// 4.调用 getName
jstring getNameResult = static_cast<jstring>(env->CallObjectMethod(student, getName));
const char * getNameValue = env->GetStringUTFChars(getNameResult, NULL);
LOGE("调用到getName方法，值是:%s\n", getNameValue);
```

#### 5. 类型转换

* Java String转换为std::string
* std::map与Java HashMap互转

```c++
std::string JavaToStdString(JNIEnv* jni, const jstring& j_string)
{
  const jclass string_class = GetObjectClass(jni, j_string);
  const jmethodID get_bytes = GetMethodID(jni, string_class, "getBytes", "(Ljava/lang/String;)[B");
  const jstring charset_name = jni->NewStringUTF("ISO-8859-1");
  const jbyteArray j_byte_array = (jbyteArray)jni->CallObjectMethod(j_string, get_bytes, charset_name);
  const size_t len = jni->GetArrayLength(j_byte_array);
  std::vector<char> buf(len);
  jni->GetByteArrayRegion(j_byte_array, 0, len, reinterpret_cast<jbyte*>(&buf[0]));
  return std::string(buf.begin(), buf.end());
}

// 用这个
std::string JNIEnvironment::JavaToStdString(const jstring& j_string) {
    const char* jchars = jni_->GetStringUTFChars(j_string, nullptr);
    CHECK_EXCEPTION(jni_, "Error during GetStringUTFChars");
    const int size = jni_->GetStringUTFLength(j_string);
    CHECK_EXCEPTION(jni_, "Error during GetStringUTFLength");
    std::string ret(jchars, size);
    jni_->ReleaseStringUTFChars(j_string, jchars);
    CHECK_EXCEPTION(jni_, "Error during ReleaseStringUTFChars");
    return ret;
}

jobject StdMapStringToJava(std::map<std::string, std::string> additional_http_headers)
{
    jclass c = jni->FindClass("java/util/HashMap");
    jobject o = jni->NewObject(c, jni->GetMethodID(c, "<init>", "()V"));
    jmethodID m = jni->GetMethodID(c, "put", "(Ljava/lang/Object;Ljava/lang/Object;)Ljava/lang/Object;");
    for (const auto& it : additional_http_headers) {
        jni->CallObjectMethod(o, m, jni->NewStringUTF(it.first.c_str()), jni->NewStringUTF(it.second.c_str())); 
    }
    return o;
}

std::map<std::string, std::string> JavaToStdMapStrings(JNIEnv* jni, jobject j_map)
{
    jclass map_class = jni->FindClass("java/util/Map");
    jclass set_class = jni->FindClass("java/util/Set");
    jclass iterator_class = jni->FindClass("java/util/Iterator");
    jclass entry_class = jni->FindClass("java/util/Map$Entry");
    jmethodID entry_set_method = jni->GetMethodID(map_class, "entrySet", "()Ljava/util/Set;");
    jmethodID iterator_method = jni->GetMethodID(set_class, "iterator", "()Ljava/util/Iterator;");
    jmethodID has_next_method = jni->GetMethodID(iterator_class, "hasNext", "()Z");
    jmethodID next_method = jni->GetMethodID(iterator_class, "next", "()Ljava/lang/Object;");
    jmethodID get_key_method = jni->GetMethodID(entry_class, "getKey", "()Ljava/lang/Object;");
    jmethodID get_value_method = jni->GetMethodID(entry_class, "getValue", "()Ljava/lang/Object;");

    jobject j_entry_set = jni->CallObjectMethod(j_map, entry_set_method);
    jobject j_iterator = jni->CallObjectMethod(j_entry_set, iterator_method);

    std::map<std::string, std::string> result;
    while (jni->CallBooleanMethod(j_iterator, has_next_method)) {
        jobject j_entry = jni->CallObjectMethod(j_iterator, next_method);
        jstring j_key = static_cast<jstring>(jni->CallObjectMethod(j_entry, get_key_method));
        jstring j_value = static_cast<jstring>(jni->CallObjectMethod(j_entry, get_value_method));
        result[JavaToStdString(jni, j_key)] = JavaToStdString(jni, j_value);
    }

    return result;
}
```

#### 6. 本地方法动态注册

结合引用的封装。示例：

```C++
extern "C" int jniRegisterNativeMethods_C(C_JNIEnv* env, const char* className,
                                          const JNINativeMethod* gMethods, int numMethods)
{
    JNIEnv* e = reinterpret_cast<JNIEnv*>(env);
    LOGV("Registering %s's %d native methods...", className, numMethods);
    scoped_local_ref<jclass> c(env, findClass(env, className));
    if (c.get() == NULL) {
        char* tmp;
        const char* msg;
        if (asprintf(&tmp,
                     "Native registration unable to find class '%s'; aborting...",
                     className) == -1) {
            // Allocation failed, print default warning.
            msg = "Native registration unable to find class; aborting...";
        } else {
            msg = tmp;
        }
        e->FatalError(msg);
    }
    if ((*env)->RegisterNatives(e, c.get(), gMethods, numMethods) < 0) {
        char* tmp;
        const char* msg;
        if (asprintf(&tmp, "RegisterNatives failed for '%s'; aborting...", className) == -1) {
            // Allocation failed, print default warning.
            msg = "RegisterNatives failed; aborting...";
        } else {
            msg = tmp;
        }
        e->FatalError(msg);
    }
    return 0;
}
```

可再做一层封装，给外部规范化调用。



动态注册和jni默认的静态注册（比如生成的Java_com_insta360_voiceassist_recognition_VoiceProcess_startProcess）区别：

可以简单这样理解：比如有个Programmer类，静态注册相当于每调用一个方法，都要new出来一个对象，比如

new Programmer().getUp()

new Programmer().coding()

new Programmer().sleep()

动态注册相当于：
Programmer a = new Programmer();

a.getUp();

a.coding();

a.sleep();

缺点是动态注册必须严格写好函数的每个参数的签名，且java层的方法无法直接通过Android Studio链接跳转到C++的方法，需要手动搜索。



#### 7. 其他（视需求决定是否封装）

* Jni静态缓存
* 签名

我们在调用这类<font color="red">Get&lt;Static&gt;MethodID、Get&lt;Static&gt;&lt;Type&gt;Field、Call&lt;Static&gt;&lt;Type&gt;Method</font>接口的时候都需要填入signature，signature用于表示描述Java类型对应C/C++类型。基本类型使用单字符表示，结构体使用L + 包名 + 结构名 + ;表示，因为JNI需要知道结构体的完整包名才能找到对应的类型。

| Java 类型   | Native 类型          | 类型大小                                       | 签名                                                                                                     |
|:--------- |:------------------ |:------------------------------------------ |:------------------------------------------------------------------------------------------------------ |
| boolean   | jboolean / uint8_t | unsigned 8 bits                            | Z                                                                                                      |
| byte      | jbyte / int8_t     | signed 8 bits                              | B                                                                                                      |
| char      | jchar / uint16_t   | unsigned 16 bits                           | C                                                                                                      |
| short     | jshort / int16_t   | signed 16 bits                             | S                                                                                                      |
| int       | jint / int32_t     | signed 32 bits                             | I                                                                                                      |
| long      | jlong / int64_t    | signed 64 bits                             | J                                                                                                      |
| float     | jfloat / float     | 32 bits                                    | F                                                                                                      |
| double    | jdouble / double   | 64 bits                                    | D                                                                                                      |
| void      | void               | N/A                                        | V                                                                                                      |
| Object    | jobject            | 引用对象大小，包括 jclass/jstring/jarray/jthrowable | Lfully/qualified/class/name;<br />比如：Ljava/nio/ByteBuffer;<br />JLandroid/media/MediaCodec$BufferInfo; |
| String    | jstring / c++对象类   | N/A                                        | Ljava/lang/String;                                                                                     |
| Object[]  | jobjectArray       | N/A                                        | N/A                                                                                                    |
| boolean[] | jbooleanArray      | N/A                                        | [Z                                                                                                     |
| byte[]    | jbyteArray         | N/A                                        | [B                                                                                                     |
| char[]    | jcharArray         | N/A                                        | [C                                                                                                     |
| short[]   | jshortArray        | N/A                                        | [S                                                                                                     |
| int[]     | jintArray          | N/A                                        | [I                                                                                                     |
| long[]    | jlongArray         | N/A                                        | [J                                                                                                     |
| float[]   | jfloatArray        | N/A                                        | [F                                                                                                     |
| double[]  | jdoubleArray       | N/A                                        | [D                                                                                                     |

### RAII思想

#### 1. 介绍

RAII（Resource Acquisition Is Initialization）,也称直译为“资源获取就是初始化”，是C++语言的一种**管理资源、避免泄漏**的机制。
C++标准保证任何情况下，已构造的对象最终会销毁，即它的析构函数最终会被调用。
RAII 机制就是利用了C++的上述特性,在需要获取使用资源RES的时候，构造一个临时对象(T)，在其构造T时获取资源，在T生命期控制对RES的访问使之始终保持有效，最后在T析构的时候释放资源。<font color="color">以达到安全管理资源对象</font>，避免资源泄漏的目的。

RAII是C++基础必备知识，非常重要，智能指针、锁都用的这个机制。

#### 2. RAII的例子

* lock_guard
  
  C++11中[lock_guard](http://www.cplusplus.com/reference/mutex/lock_guard/)对[mutex](http://www.cplusplus.com/reference/mutex/mutex/)互斥锁的管理就是典型的RAII机制，以下是C++11头文件mutex中lock_guard的源代码，看代码注释就清楚了，这是典型的RAII风格。

```C++
// @brief  Scoped lock idiom.
// Acquire the mutex here with a constructor call, then release with
// the destructor call in accordance with RAII style.
template<typename _Mutex>
class lock_guard
{
  public:
  typedef _Mutex mutex_type;

  explicit lock_guard(mutex_type& __m) : _M_device(__m)
  { _M_device.lock(); }//作者注:构造对象时加锁(申请资源),构造函数结束，就可以正常使用资源了

  lock_guard(mutex_type& __m, adopt_lock_t) : _M_device(__m)
  { } // calling thread owns mutex

  ~lock_guard()
  { _M_device.unlock(); }//作者注:析构对象时解锁(释放资源)
  // 作者注:禁用拷贝构造函数
  lock_guard(const lock_guard&) = delete;
  // 作者注:禁用赋值操作符
  lock_guard& operator=(const lock_guard&) = delete;

  private:
  mutex_type&  _M_device;
};
```

为了保证`lock_guard`对象不被错误使用，产生不可预知的后果，上面的代码中注意**`lock_guard`对象的拷贝构造函数和赋值运算符都被声明为delete(告知编译器不生成这些被标记的函数)，以确保`lock_guard`不会被复制，这是RAII机制的一个基本特征**，后面所有RAII实现都具备这个特性。

`lock_guard`的调用方式也很简单了，就借用[cplusplus.com](http://www.cplusplus.com/reference/mutex/lock_guard/)上的例程来说明吧

```C++
// lock_guard example
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::lock_guard
#include <stdexcept>      // std::logic_error

std::mutex mtx;

void print_even (int x) {
  if (x%2==0) std::cout << x << " is even\n";
  else throw (std::logic_error("not even"));
}

void print_thread_id (int id) {
  try {
    // using a local lock_guard to lock mtx guarantees unlocking on destruction / exception:
    std::lock_guard<std::mutex> lck (mtx);
    //作者注:定义一个变量lck就可以了。不用对lck作任何操作,lck在作用域结束的时候会自动释放mtx锁
    print_even(id);
  }
  catch (std::logic_error&) {
    std::cout << "[exception caught]\n";
  }
}

int main ()
{
  std::thread threads[10];
  // spawn 10 threads:
  for (int i=0; i<10; ++i)
    threads[i] = std::thread(print_thread_id,i+1);

  for (auto& th : threads) th.join();

  return 0;
}
```

### 基于RAII封装JNI

除了Java和C++层本身的内存泄露，JNI编程过程中还存在潜在的内存泄露。

JNI 中的 Local Reference 只在 native method 执行时存在，当 native method 执行完后自动失效。这种自动失效，使得对 Local Reference 的使用相对简单，native method 执行完后，它们所引用的 Java 对象的 reference count 会相应减 1。不会造成 Java Heap 中 Java 对象的内存泄漏。

而 Global Reference 对 Java 对象的引用一直有效，因此它们引用的 Java 对象会一直存在 Java Heap 中。程序员在使用 Global Reference 时，需要仔细维护对 Global Reference 的使用。如果一定要使用 Global Reference，务必确保在不用的时候删除。就像在 C 语言中，调用 malloc() 动态分配一块内存之后，调用 free() 释放一样。否则，Global Reference 引用的 Java 对象将永远停留在 Java Heap 中，造成 Java Heap 的内存泄漏。

Local Reference 在 native method 执行完成后，会自动被释放，似乎不会造成任何的内存泄漏。但这是错误的。对 Local Reference 的理解不够，会造成潜在的内存泄漏。如果创建了过多的 Local Reference，会一样导致 out of memory。实际上，nativeMethod 在运行中创建了越来越多的 JNI Local Reference，而不是看似的始终只有一个。过多的 Local Reference，导致了 JNI 内部的 JNI Local Reference 表内存溢出。

示例：

```c++
template<typename T>
class scoped_local_ref {
public:
    explicit scoped_local_ref(C_JNIEnv* env, T localRef = NULL)
            : m_env(env), m_local_ref(localRef)
    {
    }
    ~scoped_local_ref() {
        reset();
    }
    void reset(T localRef = NULL) {
        if (m_local_ref != NULL) {
            (*m_env)->DeleteLocalRef(reinterpret_cast<JNIEnv*>(m_env), m_local_ref);
            m_local_ref = localRef;
        }
    }
    T get() const {
        return m_local_ref;
    }
private:
    C_JNIEnv* const m_env;
    T m_local_ref;
    scoped_local_ref& operator=(const scoped_local_ref&) = delete;
    scoped_local_ref(const scoped_local_ref&) = delete;
};
```

**<mark>注意：</mark>** 不要所有jobject和jclass都无脑用scoped_local_ref封装。如果jni函数要给Java层返回定义的Java对象，return的jobject和jclass都不能用scoped_local_ref封装，不然会在析构时自动DeleteLocalRef，会崩溃报错：
java_vm_ext.cc:594] JNI DETECTED ERROR IN APPLICATION: expected reference of kind Local but found Global: 0x2f06

java_vm_ext.cc:594]     in call to DeleteLocalRef



### 参考文献

https://blog.csdn.net/momo0853/article/details/103977445

https://blog.csdn.net/10km/article/details/49847271

https://blog.csdn.net/10km/article/details/50147793