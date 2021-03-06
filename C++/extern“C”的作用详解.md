# 前言

如果你做过Android frameworks开发的话，或者做过jni开发的话，肯定经常看到如下代码：

```
#ifdef __cplusplus
extern "C" {
#endif
// 正式定义。。。
#ifdef __cplusplus
}
#endif
```

如果你是个C语言开发者，看到这个时可能感到疑惑，为什么要有一个extern “C”呢？



# 作用

为什么C语言开发者不常见到这种语法呢，因为这种语法其实是C++中的。

extern "C"的主要作用就是为了实现C++代码调用其他C语言代码。加上extern "C"后，会**指示编译器这部分代码按C语言的进行编译**，而不是C++的。由于C++支持函数重载，因此编译器编译函数的过程中会将函数的参数类型添加到编译后的代码中，而不仅仅是函数名；而C语言并不支持函数重载，因此编译C语言代码的函数时不会带上函数的参数类型，一般只包括函数名。

**extern “C”有两个含义：**

1. 被extern "C"限定的函数或变量是extern类型。
2. 被extern "C"修饰的变量和函数是按照C语言方式进行编译。