## 版本和调试

可在初始化地方加上如下代码，打印出详细日志：

```C++
static void LogCallback(void* ptr, int level, const char* fmt, va_list vl) //回调
{
    char temp[1024] = {0};
    vsnprintf(temp, 1024, fmt, vl);
    SLOGE("ffmpeg logs: %s", temp);
}

av_log_set_level(AV_LOG_DEBUG);
av_log_set_callback(LogCallback);
```



## Android集成

集成动态库so的CMakeList.txt如下：

```cmake

#
# libffmpegsz.so
#
#include_directories
set(SSPFFMPEG_INCLUDE ${CMAKE_CURRENT_SOURCE_DIR}/libs/include)
#so_directories
add_library(ffmpegsz SHARED IMPORTED)
include_directories(${SSPFFMPEG_INCLUDE})
set_target_properties(ffmpegsz PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/libs/${ANDROID_ABI}/libffmpegsz.so)

#
# FfmpegJni
#
set(ffmpeg_source_dir ${CMAKE_CURRENT_SOURCE_DIR}/src/main/cpp)
set(ffmpeg_headers
        ${ffmpeg_source_dir}/ffmpeg/ffmpeg_jni.h
        ${ffmpeg_source_dir}/utils/jni_utils.h
        ${ffmpeg_source_dir}/utils/scoped_local_ref.h)
set(ffmpeg_sources
        ${ffmpeg_source_dir}/ffmpeg/ffmpeg_jni.cpp
        ${ffmpeg_source_dir}/utils/jni_utils.cpp)
add_library(
        ffmpeg_jni
        SHARED
        ${ffmpeg_headers}
        ${ffmpeg_sources})
find_library(
        # Sets the name of the path variable.
        log-lib
        # Specifies the name of the NDK library that you want CMake to locate.
        log)
# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.
target_link_libraries(
        # Specifies the target library.
        ffmpeg_jni
        ffmpegsz
        # Links the target library to the log library included in the NDK.
        log
        android)
        
```

