以我下载的assimp-5.2.5为例。

## 交叉编译

先配置ANDROID_NDK_HOME环境变量。

在解压目录（CMakeLists.txt根目录）下，使用如下命令

其中-D**ANDROID_ABI** can be one of `armeabi-v7a, arm64-v8a, x86 and x86_64`

```cmake
cd /opt/assimp/
mkdir build && cd build 
cmake -DCMAKE_BUILD_TYPE=Release -DASSIMP_ANDROID_JNIIOSYSTEM=ON -DCMAKE_TOOLCHAIN_FILE=${ANDROID_NDK_HOME}/build/cmake/android.toolchain.cmake -DANDROID_NDK=${ANDROID_NDK_HOME} -DANDROID_ABI=arm64-v8a -DBUILD_SHARED_LIBS=1

make -j4
```

生成的文件会在执行命令的/bin目录下。

If you notice the library size > 100 MB, you may run the NDK strip tool to get rid of the debug symbols bundled in the library。

如果生成的so太大，那是因为带了符号表，可以用ndk命令去掉。

```shell
aarch64-linux-android-strip -g -S -d --strip-debug --verbose /opt/assimp/build/bin/libassimp.so
```



如上编译arm64-v8a的时候很顺利，但编译armeabi-v7a的时候，编译报错如下：

```shell
-- Generating done
-- Build files have been written to: /Users/jian.deng/Downloads/assimp-5.2.5
[  0%] Building CXX object port/AndroidJNI/CMakeFiles/android_jniiosystem.dir/AndroidJNIIOSystem.cpp.o
In file included from /Users/jian.deng/Downloads/assimp-5.2.5/port/AndroidJNI/AndroidJNIIOSystem.cpp:59:
/Users/jian.deng/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/c++/v1/fstream:950:9: error: 
      use of undeclared identifier 'fseeko'
    if (fseeko(__file_, __width > 0 ? __width * __off : 0, __whence))
        ^
/Users/jian.deng/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/c++/v1/fstream:213:5: note: 
      in instantiation of member function 'std::__ndk1::basic_filebuf<char, std::__ndk1::char_traits<char> >::seekoff' requested here
    basic_filebuf();
    ^
/Users/jian.deng/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/c++/v1/fstream:1358:14: note: 
      in instantiation of member function 'std::__ndk1::basic_filebuf<char, std::__ndk1::char_traits<char> >::basic_filebuf' requested here
    explicit basic_ofstream(const char* __s, ios_base::openmode __mode = ios_base::out);
             ^
/Users/jian.deng/Downloads/assimp-5.2.5/port/AndroidJNI/AndroidJNIIOSystem.cpp:171:23: note: in instantiation of member function
      'std::__ndk1::basic_ofstream<char, std::__ndk1::char_traits<char> >::basic_ofstream' requested here
        std::ofstream assetExtracted(newPath.c_str(), std::ios::out | std::ios::binary);
                      ^
In file included from /Users/jian.deng/Downloads/assimp-5.2.5/port/AndroidJNI/AndroidJNIIOSystem.cpp:59:
/Users/jian.deng/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/c++/v1/fstream:952:20: error: 
      use of undeclared identifier 'ftello'
    pos_type __r = ftello(__file_);
                   ^
/Users/jian.deng/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/c++/v1/fstream:968:9: error: 
      use of undeclared identifier 'fseeko'
    if (fseeko(__file_, __sp, SEEK_SET))
        ^
/Users/jian.deng/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/c++/v1/fstream:213:5: note: 
      in instantiation of member function 'std::__ndk1::basic_filebuf<char, std::__ndk1::char_traits<char> >::seekpos' requested here
    basic_filebuf();
    ^
/Users/jian.deng/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/c++/v1/fstream:1358:14: note: 
      in instantiation of member function 'std::__ndk1::basic_filebuf<char, std::__ndk1::char_traits<char> >::basic_filebuf' requested here
    explicit basic_ofstream(const char* __s, ios_base::openmode __mode = ios_base::out);
             ^
/Users/jian.deng/Downloads/assimp-5.2.5/port/AndroidJNI/AndroidJNIIOSystem.cpp:171:23: note: in instantiation of member function
      'std::__ndk1::basic_ofstream<char, std::__ndk1::char_traits<char> >::basic_ofstream' requested here
        std::ofstream assetExtracted(newPath.c_str(), std::ios::out | std::ios::binary);
                      ^
In file included from /Users/jian.deng/Downloads/assimp-5.2.5/port/AndroidJNI/AndroidJNIIOSystem.cpp:59:
/Users/jian.deng/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/c++/v1/fstream:1032:13: error: 
      use of undeclared identifier 'fseeko'
        if (fseeko(__file_, -__c, SEEK_CUR))
            ^
/Users/jian.deng/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/c++/v1/fstream:213:5: note: 
      in instantiation of member function 'std::__ndk1::basic_filebuf<char, std::__ndk1::char_traits<char> >::sync' requested here
    basic_filebuf();
    ^
/Users/jian.deng/Library/Android/sdk/ndk/21.4.7075529/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/c++/v1/fstream:1358:14: note: 
      in instantiation of member function 'std::__ndk1::basic_filebuf<char, std::__ndk1::char_traits<char> >::basic_filebuf' requested here
    explicit basic_ofstream(const char* __s, ios_base::openmode __mode = ios_base::out);
             ^
/Users/jian.deng/Downloads/assimp-5.2.5/port/AndroidJNI/AndroidJNIIOSystem.cpp:171:23: note: in instantiation of member function
      'std::__ndk1::basic_ofstream<char, std::__ndk1::char_traits<char> >::basic_ofstream' requested here
        std::ofstream assetExtracted(newPath.c_str(), std::ios::out | std::ios::binary);
                      ^
4 errors generated.
```

google发现`fseeko64` is not available until android-24. You have to either raise your `minSdkVersion` or stop using `_FILE_OFFSET_BITS=64`.



解决方案为在CMakeLists.txt的第一行加上set(ANDROID_PLATFORM 24)。然后编译成功。



## Android集成

根据官网https://assimp-docs.readthedocs.io/en/v5.1.0/about/quickstart.html#the-android-build提示：编译so的CMakeLists.txt需要-DASSIMP_ANDROID_JNIIOSYSTEM=ON



注意安卓引入的CMakeLists.txt最后的target_link_libraries要引入assimp，不然可能出现如下报错：

error: undefined reference to 'Assimp::Importer::Importer()'

参考CMakeLists.txt如下：

```cmake
cmake_minimum_required(VERSION 3.4.1)

# Declares and names the project.
project("switchablegapi")

add_definitions(-DSTBI_ONLY_PNG -DANDROID_PLATFORM) # -DUSE_VULKAN

# build native_app_glue as a static lib
set(APP_GLUE_DIR ${ANDROID_NDK}/sources/android/native_app_glue)
set(APP_GLM_DIR ${CMAKE_CURRENT_SOURCE_DIR}/switchablegapi/third-party/glm)
set(APP_ASSIMP_DIR ${CMAKE_CURRENT_SOURCE_DIR}/switchablegapi/third-party/assimp_5_2_5/include/assimp)
include_directories(
        ${APP_GLUE_DIR}
        ${APP_GLM_DIR}
        ${APP_ASSIMP_DIR}
)
add_library(app-glue STATIC ${APP_GLUE_DIR}/android_native_app_glue.c)

# assimp
add_library(assimp SHARED IMPORTED)
set_target_properties(assimp PROPERTIES
        IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/../../../libs/${ANDROID_ABI}/libassimp.so)
        
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Werror \
                     -DVK_USE_PLATFORM_ANDROID_KHR")
                     
add_library( # Sets the name of the library.
        youProjName
        # Sets the library as a shared library.
        SHARED
        # Provides a relative path to your source file(s).
        ${your_proj_headers}
        ${your_proj_sources}
)
                     
# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.
find_library( # Sets the name of the path variable.
        log-lib
        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.
target_link_libraries( # Specifies the target library.
        switchablegapi

        android
        app-glue

        assimp

        GLESv3
        EGL

        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})
```



## 参考文档

https://medium.com/xrpractices/build-assimp-for-android-3651eed94ac

https://stackoverflow.com/questions/55571842/android-error-use-of-undeclared-identifier-fseeko

https://stackoverflow.com/questions/62327296/cross-compile-for-android-on-linux-with-android-api-predefinition-fails-my-p

https://blog.csdn.net/zhuxiaoyang2000/article/details/107427970