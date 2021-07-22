### 一、 基本操作

##### 1. 对于这样一个Android.mk文件：

```makefile
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE    := TwoDG1
LOCAL_SRC_FILES := Triangle.cpp Square.cpp TwoDG1.cpp

LOCAL_LDLIBS := -lGLESv1_CM -llog

include $(BUILD_SHARED_LIBRARY)
```



##### 2. 对应的CMakeList.txt

```makefile
add_library( # Sets the name of the library.
             TwoDG1
             # Sets the library as a shared library.
             SHARED
             # Provides a relative path to your source file(s).
             src/main/cpp/native-lib.cpp
             src/main/cpp/Triangle.cpp
             src/main/cpp/Square.cpp)

find_library( # Sets the name of the path variable.
              log-lib
              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log)

#add lib dependencies
target_link_libraries( # Specifies the target library.
                       TwoDG1
                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib}
                       GLESv1_CM)
```



### 二、引入.so或.a

##### 1. Android.mk

```makefile
TEMP_LOCAL_PATH :=$(call my-dir)

include $(CLEAR_VARS)  
LOCAL_MODULE := xlog
LOCAL_MODULE_FILENAME := libmarsxlog
LOCAL_SRC_FILES := $(TEMP_LOCAL_PATH)/../../../../../toolkit/libs/${TARGET_ARCH_ABI}/libmarsxlog.so
LOCAL_SRC_FILES := $(LOCAL_SRC_FILES:$(LOCAL_PATH)/%=%)

LOCAL_EXPORT_C_INCLUDES := $(TEMP_LOCAL_PATH)

include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := stl
LOCAL_MODULE_FILENAME := libstlport_shared
LOCAL_SRC_FILES := $(TEMP_LOCAL_PATH)/../../../../../toolkit/libs/${TARGET_ARCH_ABI}/libstlport_shared.so

include $(PREBUILT_SHARED_LIBRARY)
```



##### 2. CMakeList.txt

```makefile
# Sets the minimum version of CMake required to build the native library.
cmake_minimum_required(VERSION 3.4.1)

#
# stl
#
add_library(stlport_shared SHARED IMPORTED)
set_target_properties(stlport_shared
        PROPERTIES IMPORTED_LOCATION
        ${CMAKE_CURRENT_SOURCE_DIR}/../toolkit/libs/${ANDROID_ABI}/libstlport_shared.so)

#
# xLog
#
set(xlog_source_dir ${CMAKE_CURRENT_SOURCE_DIR}/src/main/cpp/xlogger)
add_library(marsxlog SHARED IMPORTED)
include_directories(${xlog_source_dir})
set_target_properties(marsxlog
        PROPERTIES IMPORTED_LOCATION
        ${CMAKE_CURRENT_SOURCE_DIR}/../toolkit/libs/${ANDROID_ABI}/libmarsxlog.so)

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
        sonic-lib
        stlport_shared
        marsxlog
        # Links the target library to the log library included in the NDK.
        log
        android)
```



### 三、对比注意事项

Android.mk里是	${TARGET_ARCH_ABI}

CMakeLists.txt里是	${ANDROID_ABI}



### 四、CMakeList.txt规范

一个规范的CMakeFile.txt是可以让别的开发通过这个快速了解工程结构的，因此也要尽量写规范。示例代码如下：

```makefile
# Sets the minimum version of CMake required to build the native library.
cmake_minimum_required(VERSION 3.4.1)

#
# stl
#
add_library(stlport_shared SHARED IMPORTED)
set_target_properties(stlport_shared
        PROPERTIES IMPORTED_LOCATION
        ${CMAKE_CURRENT_SOURCE_DIR}/../toolkit/libs/${ANDROID_ABI}/libstlport_shared.so)

#
# xLog
#
set(xlog_source_dir ${CMAKE_CURRENT_SOURCE_DIR}/src/main/cpp/xlogger)
add_library(marsxlog SHARED IMPORTED)
include_directories(${xlog_source_dir})
set_target_properties(marsxlog
        PROPERTIES IMPORTED_LOCATION
        ${CMAKE_CURRENT_SOURCE_DIR}/../toolkit/libs/${ANDROID_ABI}/libmarsxlog.so)

#
# libandroid_audio_mixer.so
#
set(android_audio_mixer_source_dir ${CMAKE_CURRENT_SOURCE_DIR}/src/main/cpp/androidmixer)
add_library(android_audio_mixer SHARED IMPORTED)
# Specifies dependent .h dir
include_directories(${android_audio_mixer_source_dir})  # method 1
#target_include_directories( # method 2
#        audiomixer-lib
#        PRIVATE
#        ${CMAKE_CURRENT_SOURCE_DIR}/libs/include)   # ${audio_mixer_source_dir}
set_target_properties(android_audio_mixer
        PROPERTIES IMPORTED_LOCATION
        ${CMAKE_CURRENT_SOURCE_DIR}/../streaming/src/main/jniLibs/${ANDROID_ABI}/libandroid_audio_mixer.so)


#
# Sonic
#
set(sonic_source_dir ${CMAKE_CURRENT_SOURCE_DIR}/src/main/cpp/sonic)
add_library(sonic-lib
        SHARED
        ${sonic_source_dir}/sonic.c
        ${sonic_source_dir}/sonicjni.c)

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
        sonic-lib
        stlport_shared
        marsxlog
        # Links the target library to the log library included in the NDK.
        log
        android)

#
# png
#
set(png_source_dir ${CMAKE_CURRENT_SOURCE_DIR}/src/main/cpp/libpng)
set(png_headers
        ${png_source_dir}/pnginfo.h
        ${png_source_dir}/config.h
        ${png_source_dir}/png.h
        ${png_source_dir}/pngconf.h
        ${png_source_dir}/pngdebug.h
        ${png_source_dir}/pnglibconf.h
        ${png_source_dir}/pngpriv.h
        ${png_source_dir}/pngstruct.h)

set(png_sources
        ${png_source_dir}/arm/arm_init.c
        ${png_source_dir}/arm/filter_neon.S
        ${png_source_dir}/arm/filter_neon_intrinsics.c
        ${png_source_dir}/png.c
        ${png_source_dir}/pngerror.c
        ${png_source_dir}/pngget.c
        ${png_source_dir}/pngmem.c
        ${png_source_dir}/pngpread.c
        ${png_source_dir}/pngread.c
        ${png_source_dir}/pngrio.c
        ${png_source_dir}/pngrtran.c
        ${png_source_dir}/pngrutil.c
        ${png_source_dir}/pngset.c
        ${png_source_dir}/pngtrans.c
        ${png_source_dir}/pngwio.c
        ${png_source_dir}/pngwrite.c
        ${png_source_dir}/pngwtran.c
        ${png_source_dir}/pngwutil.c)

add_library(png
        STATIC
        ${png_sources}
        ${png_headers})

target_link_libraries(png -lz)

#
# apng_drawbale
#
set(apng_drawable_source_dir ${CMAKE_CURRENT_SOURCE_DIR}/src/main/cpp/apng-drawbale)
set(apng_drawbale_headers
        ${apng_drawable_source_dir}/ApngDecoder.h
        ${apng_drawable_source_dir}/ApngFrame.h
        ${apng_drawable_source_dir}/ApngImage.h
        ${apng_drawable_source_dir}/Error.h
        ${apng_drawable_source_dir}/Log.h
        ${apng_drawable_source_dir}/StreamSource.h)

set(apng_drawbale_sources
        ${apng_drawable_source_dir}/ApngDecoder.cpp
        ${apng_drawable_source_dir}/ApngDecoderJni.cpp
        ${apng_drawable_source_dir}/ApngFrame.cpp
        ${apng_drawable_source_dir}/ApngImage.cpp
        ${apng_drawable_source_dir}/StreamSource.cpp)

add_library(apng-drawable
        SHARED
        ${apng_drawbale_headers}
        ${apng_drawbale_sources})
# Specifies dependent libpng dir
target_include_directories(
        apng-drawable
        PUBLIC
        ${png_source_dir})

if (${CMAKE_BUILD_TYPE} MATCHES DEBUG)
    find_library(
            log-lib
            log)

    target_link_libraries(
            apng-drawable
            ${log-lib}
            -ljnigraphics
            png)
else (${CMAKE_BUILD_TYPE} MATCHES DEBUG)
    target_link_libraries(
            apng-drawable
            -ljnigraphics
            png)
endif (${CMAKE_BUILD_TYPE} MATCHES DEBUG)

#
# AudioMixer
#
set(audio_mixer_source_dir ${CMAKE_CURRENT_SOURCE_DIR}/src/main/cpp/audiomixer)
set(audio_mixer_headers
        ${audio_mixer_source_dir}/audiomixer-jni.h)
set(audio_mixer_sources
        ${audio_mixer_source_dir}/audiomixer-jni.cpp)
add_library(
        audiomixer-lib
        SHARED
        ${audio_mixer_headers}
        ${audio_mixer_sources})

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
        audiomixer-lib
        android_audio_mixer
        # Links the target library to the log library included in the NDK.
        log
        android)
```







