# 一、概念

### 1. JavaVm

JavaVM 是虚拟机在 JNI 层的代表，**一个进程只有一个 JavaVM**，所有的线程共用一个 JavaVM。



### 2. JNIEnv

JNIEnv 表示 Java 调用 native 语言的环境，是一个封装了几乎全部 JNI 方法的指针。

JNIEnv **只在创建它的线程生效，不能跨线程传递**，不同线程的 JNIEnv 彼此独立。

native 环境中创建的线程，如果需要访问 JNI，必须要调用 AttachCurrentThread 关联，并使用 DetachCurrentThread 解除链接。



# 二、两种代码风格（C/C++）

JavaVM 和 JNIEnv 在 C 语言环境下和 C++ 环境下调用是有区别的，主要表现在：

> C风格：(*env)->NewStringUTF(env, “Hellow World!”);
>
> C++风格：env->NewStringUTF(“Hellow World!”);

建议使用 C++ 风格，这也是大部分代码使用的形式。

注意：**C++ 风格其实只是封装了 C 风格，使得调用更加简介方便**。



# 三、定义

### 1. JavaVM 和 JNIEnv 在中的定义如下：

```c++
#if defined(__cplusplus)
typedef _JNIEnv JNIEnv;
typedef _JavaVM JavaVM;
#else
typedef const struct JNINativeInterface* JNIEnv;
typedef const struct JNIInvokeInterface* JavaVM;
#endif
```

这里分了 C 和 C++。如果是 C++ 环境下，则只是对 _JNIEnv 和 _JavaVM 的一个重命名；如果是 C 环境下，则是指向 JNINativeInterface 结构体和 JNIInvokeInterface 结构体的指针。



### 2. 继续看 JNINativeInterface 结构体和 JNIInvokeInterface 结构体的定义

```c++
struct JNINativeInterface {
    ...
    jint        (*GetVersion)(JNIEnv *);
    jclass      (*DefineClass)(JNIEnv*, const char*, jobject, const jbyte*,
                        jsize);
    jclass      (*FindClass)(JNIEnv*, const char*);
    ...
};


struct JNIInvokeInterface {
    void*       reserved0;
    void*       reserved1;
    void*       reserved2;

    jint        (*DestroyJavaVM)(JavaVM*);
    jint        (*AttachCurrentThread)(JavaVM*, JNIEnv**, void*);
    jint        (*DetachCurrentThread)(JavaVM*);
    jint        (*GetEnv)(JavaVM*, void**, jint);
    jint        (*AttachCurrentThreadAsDaemon)(JavaVM*, JNIEnv**, void*);
};
```

出于篇幅考虑，这里省略了一下 JNINativeInterface 结构体，它里面还包含了大量的方法。

所以我们可以知道，**C 风格下的 JavaVM 和 JNIEnv 就是指向的这两个结构体的指**针，通过这两个指针我们可以调用到这两个结构体里的各个方法。那么 C++ 风格下的 _JNIEnv 和 _JavaVM 又是怎么定义的呢？



### 3. _JNIEnv 和 _JavaVM

```c++
struct _JNIEnv {
    /* do not rename this; it does not seem to be entirely opaque */
    const struct JNINativeInterface* functions;

#if defined(__cplusplus)

    jint GetVersion()
    { return functions->GetVersion(this); }

    jclass DefineClass(const char *name, jobject loader, const jbyte* buf,
        jsize bufLen)
    { return functions->DefineClass(this, name, loader, buf, bufLen); }

    jclass FindClass(const char* name)
    { return functions->FindClass(this, name); }

    ...

#endif /*__cplusplus*/
};


struct _JavaVM {
    const struct JNIInvokeInterface* functions;

#if defined(__cplusplus)
    jint DestroyJavaVM()
    { return functions->DestroyJavaVM(this); }
    jint AttachCurrentThread(JNIEnv** p_env, void* thr_args)
    { return functions->AttachCurrentThread(this, p_env, thr_args); }
    jint DetachCurrentThread()
    { return functions->DetachCurrentThread(this); }
    jint GetEnv(void** env, jint version)
    { return functions->GetEnv(this, env, version); }
    jint AttachCurrentThreadAsDaemon(JNIEnv** p_env, void* thr_args)
    { return functions->AttachCurrentThreadAsDaemon(this, p_env, thr_args); }
#endif /*__cplusplus*/
};
```

这里还是省略了很多 _JNIEnv 里的方法。

通过上面代码我们可以看到，**_JNIEnv 和 _JavaVM 其实只是对 JNINativeInterface 和 JNIInvokeInterface 结构体的一层封装**，实际调用和操作的还是 JNINativeInterface 和 JNIInvokeInterface 里的方法。



# 四、总结

以上，我们可以简单的了解和认识到 JavaVM 和 JNIEnv 在 JNI 开发中扮演的角色，我们 JNI 的绝大多数操作都是**通过这两者来调用到 JNINativeInterface 和 JNIInvokeInterface 结构体里的相关方法**。

**其中 JavaVM 是一个全局变量，一个进程只有一个 JavaVM 对象。**

**而 JNIEnv 是一个线程拥有一个，不同线程的 JNIEnv 彼此独立。**

最后由于 JavaVM 和 JNIEnv 在 C/C++ 环境下的实现不同，所以产生了两种代码风格，即：

> C风格：(*env)->NewStringUTF(env, “Hellow World!”);
>
> C++风格：env->NewStringUTF(“Hellow World!”);



————————————————

版权声明：本文为CSDN博主「阿飞__」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/afei__/article/details/80986203