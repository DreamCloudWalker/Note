## 1. loader.loadNative("c++_shared"); 

所有是c++的so load之前，先加载这个。



## 2. 不同地方编译的so，一起用的时候，NDK版本要保持一致

可以在local.properties里指定ndk.dir，比如：

```groovy
ndk.dir=/Users/jian.deng/Library/Android/sdk/ndk/21.4.7075529
```

也可以再项目跟目录的build.gradle中指定，比如：

```groovy
android {
    compileSdkVersion 30
    buildToolsVersion "30.0.3"

    ndkVersion "21.4.7075529" // 必须使用此版本编译打包。保持和其他团队统一

    defaultConfig {
        minSdkVersion 16
        targetSdkVersion 30
        versionCode 1
        versionName "1.0"
```

使用NDK时，你可能会倾向于使用最新的编译平台，但事实上这是错误的，<font color="red">因为NDK平台不是后向兼容（兼容过去的版本）的，而是前向兼容（兼容将来的版本）的</font>。比如你的库是用ndk21打的，别的团队的so是ndk23打的。如果别的团队的so先加载起来，如果你的so里用了一些C++的方法比如stringstream，就可能出现莫名其妙的崩溃。

