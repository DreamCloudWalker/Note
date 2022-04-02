## dlopen failed: cannot locate symbol "XXXX" xxxx.so

不同链接方式时，dlopen会打开指定的系统中(手机中)或提供的动态库，并使用 dlsym 获取符号地址，也就是说，如果，在此时的手机中如果找不到，那么就会出问题，一般和 API 有关系。

人为因素就是，编译这个 .so 库的人，他在编译的时候没考虑到下面这些情况，导致提供给别人用的时候，或者自己用的时候在高 API 版本手机出现问题。 　　



## NDK兼容性

使用NDK时，你可能会倾向于使用最新的编译平台，但事实上这是错误的，<font color="red">因为NDK平台不是后向兼容（兼容过去的版本）的，而是前向兼容（兼容将来的版本）的</font>。推荐使用app的minSdkVersion对应的编译平台。





## 参考文档

https://bleoo.github.io/2017/08/31/Android-5-0%E5%88%B0Android-6-0-%E7%9A%84so%E5%8A%A8%E6%80%81%E5%BA%93%E7%9A%84%E9%80%82%E9%85%8D%E9%97%AE%E9%A2%98/

