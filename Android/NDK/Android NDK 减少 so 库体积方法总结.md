## 1. 背景

基于亚马逊 AVS Device SDK 改造的全链路语音 SDK 最终编译的动态库有几十个，单架构动态库大小有几十兆，之前在Iot设备中勉强跑着，但是这个体积对于手机应用来说是致命的，各个模块费事费力能优化个几K的体积就不错了，我这直接给上个几十兆的，APP平台方肯定无法接受。

但是一是有业务需求，二是自己又想把SDK推到手机APP，提高用户量，验证SDK的稳定性和交互体验，所以开始了漫长的瘦身过程，最后单架构压缩到了五兆一下，虽然还是有点大，但是比起之前有了很大的提升。



## 2. 删除无用模块

AVS Device SDK是主要应用在音响的控制台程序，而且代码是跨平台的，所以一是有很多为了跨平台做的冗余，二是有很多我们根本用不到的模块。

比如为了做本地存储引入了一个Sqlite的动态库，我们本身也用不到本地存储，像闹钟设置之类的放到APP层即可，而且就算是需要存储也完全可以使用Android和iOS平台提供的Sqlite。删除用不到的模块是包体积优化空间最大最快的。



## 3. 第三方库替换为Android/iOS平台提供能力

AVS Device SDK在Android平台基于ffmpeg做解码实现了音频播放器，对于我们的场景主要使用用播放器来播放TTS，而TTS是和服务协商好固定的mp3格式，完全没有必要为了一个mp3解码引入一个庞大的ffmpeg库。

这里我们使用Android平台提供的Jni层的媒体库来做音频解码。而且即使是Android平台JNI层不支持，也可以单独依赖一个mp3解码库，而不是庞大的ffmpeg。对于整个包体积来说，第三方模块往往相对来说是比较大的。



## 4. 使用strip

使用NDK toolchain可以把调试的C++ 符号表(Symbol Table)中数据删除，我们一般我们打成APK会自动帮我们做这个工作，当然也可以手动设置：

手动的在链接选项中加入 strip参数，配置如下所示：

```
SET_TARGET_PROPERTIES(yoga PROPERTIES LINK_FLAGS "-Wl,-s")
```

也可以手动执行ndk提供的`aarch64-linux-android-strip`命令移除动态库中的调试信息，这种方式除了前面方法外优化体积最高的方式，比如libLibSampleApp.so从48M直接优化到了992k。



## 4. 设置编译器的优化flag

编译器有个优化flag可以设置，分别是-Os（体积最小），-O3(性能最优)等。这里将编译器的优化flag设置为-Os，以便减少体积。

CMake:

```
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Os")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS}")
```

Android.mk

```
LOCAL_CPPFLAGS += -Os
LOCAL_CFLAGS += -Os
```

除了直接删除占用体积较大的模块外，编译器优化是排下来优化空间最大的方法。设置完`-Os`后占用提交较大的前几个库体积对比：

| 库名                | 优化前体积 | 优化后体积 |
| :------------------ | :--------- | :--------- |
| libLibSampleApp.so  | 48M        | 33M        |
| libAVSCommon.so     | 28M        | 22M        |
| libDefaultClient.so | 14M        | 9.9M       |



## 5. 使用 gc-sections去除没有用到的函数

有些时候代码量比较大的时候我们没办法手动发现无用的函数，这个时候可以可以开启编译器的gc-sections选项，让编译器自动的帮你做到这一点。

编译器可以配置自动去除未使用的函数和变量，以下是配置方式：

CMake:

```
# 去除未使用函数与变量
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ffunction-sections -fdata-sections")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS}")
# 设置去除未使用代码的链接flag
SET_TARGET_PROPERTIES(yoga PROPERTIES LINK_FLAGS "-Wl,--gc-sections")        
```

Android.mk:

```
OCAL_CPPFLAGS += -ffunction-sections -fdata-sections
LOCAL_CFLAGS += -ffunction-sections -fdata-sections 
LOCAL_LDFLAGS += -Wl,--gc-sections
```



## 6. 设置编译器的 Visibility Feature

Visibility Feature就是用来控制在哪些函数可以在符号表中被输入，由于C++并不是完全面向对象的，非类的方法并没有public这种修饰符，因此，要用Visibility Feature来控制哪些函数可以被外部调用。而JNI提供了一个宏-JNIEXPORT来控制这点。所以只要对函数加上这个宏，像这样：

```
// JNIEXPORT就是控制可见的宏
// JNICALL在NDK这里没有什么意义，只是个标识宏
JNIEXPORT void JNICALL Java_ClassName_MethodName(JNIEnv *env, jobject obj, jstring javaString)
```

然后在编译器的FLAGS选项开启 -fvisibility = hidden 就可以。这样，不仅可以控制函数的可见性，并且可以减少包体的大小。



## 7. 去除C++代码中的iostream等直接IO相关代码

使用STL中的iostream相关库会明显的增加包的体积，而Android本身是有预编译库(android/log.h)可以代替输入到控制台的工具的。在我们的SDK中由于之前是控制台程序所以用到了输入输出，编译的时候没有把这块排除出去，造成了一定的体积冗余。



## 8. STL的使用方式

对于C++的library，引用方式有2种：

- 静态方式(static)
- 动态方式(shared)

其中，静态方式在编译时会将用到的相关代码直接复制到目的文件中；而动态方式则会将相关的代码打成so文件，以便多次引用。由于编译器在编译时并不能知道所有被引用的地方，所以同时会打入了很多不相关的代码。

所以，如果项目中引用library的函数较多时，用动态方式可以避免多次拷贝，节省空间。相反，则直接使用静态方式会更节省空间。由于我们SDK的模块特别多，再加上整体APK里面已经有其他业务引入了动态库，所以我们用动态库的方式。



## 9. 不使用Exception和RTTI

关于这两点在网上看到的没有实践过，不过拿过来可以作为包体积持续优化的参考。

**RTTI**

**通过RTTI，能够通过基类的指针或引用来检索其所指对象的实际类型，即运行时获取对象的实际类型。C++通过下面两个操作符提供RTTI。**

（1）typeid：返回指针或引用所指对象的实际类型。

（2）dynamic_cast：将基类类型的指针或引用安全的转换为派生类型的指针或引用。

RTTI的选项是默认关闭的的，而代码中其实并没有用到相关的功能,这里可以直接关闭。

**Exception**

**使用C++的exception会增加包的大小，而目前JNI对C++的exception的支持是有bug的，比如下面这段代码就会引起程序的crash（对于低版本的android NDK）。**因此要在程序中引入exception要自己实现相关逻辑，但是这样又会增加包体大小。对于开发者来说，exception可以帮助快速定位问题，而对于使用者并不是那么重要，这里可以去掉。



## 10 总结

本文介绍了**删除无用模块，平台能力替代第三方库，使用 strip，设置编译器优化的 flag，使用gc-sections去除没有用到的函数，设置可见性，去除iostream等有助于动态库体积优化的方法。**

> 原文链接: https://juejin.cn/post/7084238491089027079



推荐阅读：

[NDK | 带你梳理 JNI 函数注册的方式和时机](http://mp.weixin.qq.com/s?__biz=MzIwNTIwMzAzNg==&mid=2654169948&idx=1&sn=06653e18e81e9f868b447fc9e1d8fe28&chksm=8cf3b86fbb8431790178ecf0d9376e34fa8d23e5dc9ecbe110a4fcf7956285bc103b7c93a082&scene=21#wechat_redirect)

[Android NDK 开发：JNI 基础篇](http://mp.weixin.qq.com/s?__biz=MzIwNTIwMzAzNg==&mid=2654168827&idx=1&sn=9ab5caa45b52033768da9f1e32583b63&chksm=8cf3b5c8bb843cde32630408b21777f196103ddfe88c5114480d5d51ae45e1b9a40279cc242e&scene=21#wechat_redirect)

[Android NDK 开发：Java 与 Native 相互调用](http://mp.weixin.qq.com/s?__biz=MzIwNTIwMzAzNg==&mid=2654168752&idx=1&sn=b3ed51f1ae6957567ad907f66fa4900d&chksm=8cf3b583bb843c95f72a874092abb745c80d5704d3a3310787031ec04589cebc637b41361e37&scene=21#wechat_redirect)

[Android NDK POSIX 多线程编程](http://mp.weixin.qq.com/s?__biz=MzIwNTIwMzAzNg==&mid=2654165578&idx=2&sn=2e2c67d9f3034f7a299dda3e2c0af8cd&chksm=8cf38979bb84006f99f194a6ae788c7d5664d1b3bdd607625250af38588835eddfe9e3dd42d6&scene=21#wechat_redirect)

[NDK 开发中 Native 方法的静态注册与动态注册](http://mp.weixin.qq.com/s?__biz=MzIwNTIwMzAzNg==&mid=2654164968&idx=1&sn=de72b2b4da414b196575340aa44ee01a&chksm=8cf384dbbb840dcd50c19b5b0ea9ee1a960bd793a8ac38f633f5606fc3c62ec6f38cb66675f2&scene=21#wechat_redirect)

[Android NDK 开发中快速定位 Crash 问题](http://mp.weixin.qq.com/s?__biz=MzIwNTIwMzAzNg==&mid=2654164504&idx=1&sn=aec751929df1a1d0b98bd5aa90ba65a4&chksm=8cf3852bbb840c3d5877fd80f93e2dba984ba591f7c427a90b159c63eb148ce2f2a7c4cb21ad&scene=21#wechat_redirect)