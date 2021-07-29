* 常见代码
    * 定义jni日志打印，其中 __VA_ARGS__ 代表 ...的可变参数
    ```c++
    #include <iostream>
    
    // 日志输出
    #include <android/log.h>
    
    #define TAG "JNISTUDY"
    // __VA_ARGS__ 代表 ...的可变参数
    #define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG, TAG,  __VA_ARGS__);
    #define LOGE(...) __android_log_print(ANDROID_LOG_ERROR, TAG,  __VA_ARGS__);
    #define LOGI(...) __android_log_print(ANDROID_LOG_INFO, TAG,  __VA_ARGS__);
    ```
* Jni对象
    * 数组操作：jintArray == int[]， jobjectArray == 引用类型对象，例如 String[]   Test[]
        * 把int[] 转成 int*： 
        ```c++
        extern "C" 
        JNIEXPORT void JNICALL
        Java_com_dengjian_testjni_MainActivity_testArraAction(JNIEnv *env, jobject thiz, jint count, jstring text_info, jintArray ints, jobjectArray strs) {
            // ① 基本数据类型  jint count， jstring text_info， 最简单的
            int countInt = count; // jint本质是int，所以可以用int接收
            LOGI("参数一 countInt:%d\n", countInt);
            
            // const char* GetStringUTFChars(jstring string, jboolean* isCopy)
            const char * textInfo = env->GetStringUTFChars(text_info, NULL);
            LOGI("参数二 textInfo:%s\n", textInfo);
            
            // ② 把int[] 转成 int*
            // jint* GetIntArrayElements(jintArray array, jboolean* isCopy)
            int* jintArray = env->GetIntArrayElements(ints, NULL);
        ```
        * Java层数组的长度
        ```c++
            // jsize GetArrayLength(jarray array) // jintArray ints 可以放入到 jarray的参数中去
            jsize size = env->GetArrayLength(ints);
            for (int i = 0; i < size; ++i) {
                *(jintArray+i) += 100; // C++的修改，影响不了Java层
                LOGI("参数三 int[]:%d\n", *jintArray+i);
            }
            /**
             * 0:           刷新Java数组，并 释放C++层数组
             * JNI_COMMIT:  只提交 只刷新Java数组，不释放C++层数组
             * JNI_ABORT:   只释放C++层数组
             */
            env->ReleaseIntArrayElements(ints, jintArray, 0);
            
            // ③：jobjectArray 代表是Java的引用类型数组，不一样
            jsize  strssize = env->GetArrayLength(strs);
            for (int i = 0; i < strssize; ++i) {
                jstring jobj = static_cast<jstring>(env->GetObjectArrayElement(strs, i));
        
                // 模糊：isCopy内部启动的机制
                // const char* GetStringUTFChars(jstring string, jboolean* isCopy)
                const char * jobjCharp = env->GetStringUTFChars(jobj, NULL);
        
                LOGI("参数四 引用类型String 具体的：%s\n", jobjCharp);
        
                // 释放jstring
                env->ReleaseStringUTFChars(jobj, jobjCharp);
            }
        }
        ```
    * 对象操作
    ```c++
    // jobject student == Student
    // jstring str  == String
    extern "C"
    JNIEXPORT void JNICALL
    Java_com_dengjian_testjni_MainActivity_putObject(JNIEnv *env, jobject thiz, jobject student, jstring str) {
        const char * strChar = env->GetStringUTFChars(str, NULL);
        LOGI("strChar：%s\n", strChar);
        env->ReleaseStringUTFChars(str, strChar);
    
        // --------------
        // 1.寻找类 Student
        // jclass studentClass = env->FindClass("com/dengjian/as_jni_project/Student"); // 第一种
        jclass studentClass =  env->GetObjectClass(student); // 第二种
    
        // 2.Student类里面的函数规则  签名
        jmethodID setName = env->GetMethodID(studentClass, "setName", "(Ljava/lang/String;)V");
        jmethodID getName = env->GetMethodID(studentClass, "getName", "()Ljava/lang/String;");
        jmethodID showInfo = env->GetStaticMethodID(studentClass, "showInfo", "(Ljava/lang/String;)V");
    
        // 3.调用 setName
        jstring value = env->NewStringUTF("AAAA");
        env->CallVoidMethod(student, setName, value);
    
        // 4.调用 getName
        jstring getNameResult = static_cast<jstring>(env->CallObjectMethod(student, getName));
        const char * getNameValue = env->GetStringUTFChars(getNameResult, NULL);
        LOGE("调用到getName方法，值是:%s\n", getNameValue);
    
        // 5.调用静态showInfo
        jstring  jstringValue = env->NewStringUTF("静态方法你好，我是C++");
        env->CallStaticVoidMethod(studentClass, showInfo, jstringValue);
    }
    ```
    * 对象进阶
    ```c++
    extern "C"
    JNIEXPORT void JNICALL
    Java_com_dengjian_testjni_MainActivity_insertObject(JNIEnv *env, jobject thiz) {
        // 1.通过包名+类名的方式 拿到 Student class  凭空拿class
        const char *studentstr = "com/dengjian/as_jni_project/Student";
        jclass studentClass = env->FindClass(studentstr);
    
        // 2.通过student的class  实例化此Student对象   C++ new Student
        jobject studentObj = env->AllocObject(studentClass); // AllocObject 只实例化对象，不会调用对象的构造函数
    
        // 方法签名的规则
        jmethodID setName = env->GetMethodID(studentClass, "setName", "(Ljava/lang/String;)V");
        jmethodID setAge = env->GetMethodID(studentClass, "setAge", "(I)V");
    
        // 调用方法
        jstring strValue = env->NewStringUTF("dengjian");
        env->CallVoidMethod(studentObj, setName, strValue);
        env->CallVoidMethod(studentObj, setAge, 99);
    
        // env->NewObject() // NewObject 实例化对象，会调用对象的构造函数
    
        // ====================  下面是 Person对象  调用person对象的  setStudent 函数等
    
        // 4.通过包名+类名的方式 拿到 Student class  凭空拿class
        const char *personstr = "com/dengjian/as_jni_project/Person";
        jclass personClass = env->FindClass(personstr);
    
        jobject personObj = env->AllocObject(personClass); // AllocObject 只实例化对象，不会调用对象的构造函数
    
        // setStudent 此函数的 签名 规则
        jmethodID setStudent = env->GetMethodID(personClass, "setStudent",
                "(Lcom/dengjian/as_jni_project/Student;)V");
    
        env->CallVoidMethod(personObj, setStudent, studentObj);
    }
    ```
    * 局部引用与全局引用
    ```c++
    jclass dogClass; // 你以为这个是全局引用，实际上他还是局部引用
    
    extern "C"
    JNIEXPORT void JNICALL
    Java_com_dengjian_testjni_MainActivity_testQuote(JNIEnv *env, jobject thiz) {
        if (NULL == dogClass) {
            /*const char * dogStr = "com/dengjian/as_jni_project/Dog";
            dogClass = env->FindClass(dogStr);*/
    
            // 升级全局引用： JNI函数结束也不释放，反正就是不释放，必须手动释放   ----- 相当于： C++ 对象 new、手动delete
            const char * dogStr = "com/dengjian/as_jni_project/Dog";
            jclass temp = env->FindClass(dogStr);
            dogClass = static_cast<jclass>(env->NewGlobalRef(temp)); // 提升全局引用
            // 记住：用完了，如果不用了，马上释放，C C++ 工程师的赞美
            env->DeleteLocalRef(temp);
        }
    
        // <init> V  是不会变的
    
        // 构造函数一
        jmethodID init = env->GetMethodID(dogClass, "<init>", "()V");
        jobject dog = env->NewObject(dogClass, init);
    
        // 构造函数2
        init = env->GetMethodID(dogClass, "<init>", "(I)V");
        dog = env->NewObject(dogClass, init, 100);
    
    
        // 构造函数3
        init = env->GetMethodID(dogClass, "<init>", "(II)V");
        dog = env->NewObject(dogClass, init, 200, 300);
    
        // 构造函数4
        init = env->GetMethodID(dogClass, "<init>", "(III)V");
        dog = env->NewObject(dogClass, init, 400, 500, 600);
    
        env->DeleteLocalRef(dog); // 释放
    }
    
    // JNI函数结束，会释放局部引用   dogClass虽然被释放，但是还不等于NULL，只是一个悬空指针而已，所以第二次进不来IF，会奔溃
    
    // 非常方便，可以使用了
    extern int age; // 声明age
    extern void show(); // 声明show函数
    
    // 手动释放全局引用
    extern "C"
    JNIEXPORT void JNICALL
    Java_com_dengjian_as_1jni_1project_MainActivity_delQuote(JNIEnv *env, jobject thiz) {
       if (dogClass != NULL) {
           LOGE("全局引用释放完毕，上面的按钮已经失去全局引用，再次点击会报错");
           env->DeleteGlobalRef(dogClass);
           dogClass = NULL; // 最好给一个NULL，指向NULL的地址，不要去成为悬空指针
       }
    
       // 测试下
       show();
    }
    ```
* JNIEXPORT和JNICALL
    * JNIEXPORT : Linux 和 Windows jni.h内部的宏定义 是不一样的，此宏代表是 对外暴露的标准形式。例如：在Windows中 对外暴露的标准已经被规定好了，所以在jni.h中的宏是以Windows对外暴露的标准规则来写的。
    * JNICALL ： Linux 和 Windows jni.h内部的宏定义 是不一样的，此宏代表是 此函数形参压栈的规则制定，例如：在Windows平台中里面的宏定义，代表函数压栈从右到左方式操作的 等等。
* CMake格式说明
```makefile
// app(build.gradle)
externalNativeBuild {
    cmake {
        // 指定CPU架构，cmake本地库，在CMakeLists.txt中的target_link_libraries中的native-lib，只会处理这里指定的平台的库。如果不写会默认处理4大平台，会更慢
        abiFilters "armeabi-v7a"
        cppFlags "-std=c++11"
    }
}

ndk {
    // 这里写了会打入apk的lib/cpu平台
    abiFilters 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
}

// CMakeLists.txt
# Step1 Sets the minimum version of CMake required to build the native library.
cmake_minimum_required(VERSION 3.4.1)

# Import the glm header file from the NDK.
add_library( glm INTERFACE )
set_target_properties( glm PROPERTIES INTERFACE_INCLUDE_DIRECTORIES ${GLM_INCLUDE})

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.
add_library(
        # Sets the name of the library.
        chestnut3d_base
        # Sets the library as a shared library.
        SHARED
        # Provides a relative path to your source file(s).
        src/main/cpp/jni_interface.cpp
        src/main/cpp/render/gl_util.cpp
        src/main/cpp/render/gl_application.cpp)

# Searches for a specified prebuilt library and stores the path as a variable.
# Because CMake includes system libraries in the search path by default,
# you only need to specify the name of the public NDK library you want to add.
# CMake verifies that the library exists before completing its build.
find_library(
        # Sets the name of the path variable.
        log-lib
        # Specifies the name of the NDK library that you want CMake to locate.
        log)

# Step , 链接具体的库，加入我们的libchestnut3d_base.so总库
# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.
target_link_libraries(
        # Specifies the target library.
        chestnut3d_base
        # Links the target library to the log library included in the NDK.
        log
        android
        GLESv2
        glm)

```
* 动态注册与静态注册
    * 静态注册优缺点：
        * 函数名长（避免C中重载）；
        * 写起来方便，调试跳转方便；
        * 效率低，执行函数时才开始初始化；
    * 动态注册：
        * 在JNI_OnLoad中去注册函数，该函数类似构造函数，在Sysytem.loadLibrary()时就会触发。比静态函数效率高。和静态相比：静态相当于是：
        ```java
        // 静态注册类似：
        new Student().xx1();
        new Student().xx2();
        new Student().xx3();
        
        // 动态注册
        Student stu = new Student();
        stu.xx1();
        stu.xx2();
        stu.xx3();
        ```
        * 动态注册代码如下：
    ```c++
    JavaVM *jVm = nullptr; // 避免不赋值出现类似0x003545 系统乱值，C++11后，取代NULL，作用是可以初始化指针赋值
    const char *mainActivityClassName = "com/dengjian/testjni/MainActivity";
    
    // native 真正的函数
    // void dynamicMethod01(JNIEnv *env, jobject thiz) { // OK的
    void dynamicMethod01() { // 也OK  如果你用不到  JNIEnv jobject ，可以不用写
        LOGD("我是动态注册的函数 dynamicMethod01...");
    }
    
    int dynamicMethod02(JNIEnv *env, jobject thiz, jstring valueStr) { // 也OK
        const char *text = env->GetStringUTFChars(valueStr, nullptr);
        LOGD("我是动态注册的函数 dynamicMethod02... %s", text);
        env->ReleaseStringUTFChars(valueStr, text);
        return 200;
    }
    
    /*
     typedef struct {
        const char* name;       // 函数名
        const char* signature;  // 函数的签名
        void*       fnPtr;      // 函数指针
     } JNINativeMethod;
     */
    static const JNINativeMethod jniNativeMethod[] = {
            {"dynamicJavaMethod01", "()V",                   (void *) (dynamicMethod01)},
            {"dynamicJavaMethod02", "(Ljava/lang/String;)I", (int *) (dynamicMethod02)},
    };
    
    
    // Java：像 Java的构造函数，如果你不写构造函数，默认就有构造函数，如果你写构造函数 覆写默认的构造函数
    // JNI JNI_OnLoad函数，如果你不写JNI_OnLoad，默认就有JNI_OnLoad，如果你写JNI_OnLoad函数 覆写默认的JNI_OnLoad函数
    extern "C"
    JNIEXPORT jint JNI_OnLoad(JavaVM *javaVm, void *) {
        // this.javaVm = javaVm;
        ::jVm = javaVm;
    
        // 做动态注册 全部做完
        JNIEnv *jniEnv = nullptr;
        int result = javaVm->GetEnv(reinterpret_cast<void **>(&jniEnv), JNI_VERSION_1_6);
    
        // result 等于0  就是成功    【C库 FFmpeg 成功就是0】
        if (result != JNI_OK) {
            return -1; // 会奔溃，故意奔溃
        }
    
        LOGE("System.loadLibrary ---》 JNI Load init");
    
        jclass mainActivityClass = jniEnv->FindClass(mainActivityClassName);
    
        // jint RegisterNatives(Class, 我们的数组==jniNativeMethod， 注册的数量 = 2)
        jniEnv->RegisterNatives(mainActivityClass,
                                jniNativeMethod,
                                sizeof(jniNativeMethod) / sizeof(JNINativeMethod));
    
        LOGE("动态 注册没有毛病");
    
        return JNI_VERSION_1_6; // AS的JDK在JNI默认最高1.6      存Java的JDKJNI 1.8
    }
    ```
* JNI多线程
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
* JNI静态缓存（OpenCV,WebRTC都大量使用了静态缓存）
```c++
public class MainActivity2 extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main); }
        static { 
            System.loadLibrary("native-lib");
        }
        // 假设这里定义了一大堆变量 
        static String name1 = "T1"; 
        static String name2 = "T2"; 
        static String name3 = "T3"; 
        static String name4 = "T4"; 
        static String name5 = "T5"; 
        static String name6 = "T6"; 
        // 省略更多 ....
        
        public static native void localCache(String name); // 普通局部缓存 弊端演示
        
        public static native void initStaticCache(); // 初始化静态缓存 
        public static native void staticCache(String name); // 静态缓存 
        public static native void clearStaticCache(); // 清除静态缓存
    
        /**
        * 静态缓存策略 */
        public void staticCacheAction(View view) { 
        // 下面是局部缓存 的演示 localCache("李元霸");
        Log.e("dengjian", "name1:" + name1);
        
        // TODO 下面是静态缓存区域 ==============================
        initStaticCache(); // 先初始化静态缓存 (注意:如果是一个类去调用，就需要在构造函数中初始化)
        
        staticCache("李白"); // 再执行... 
        Log.e("dengjian", "静态缓存区域 name1:" + name1); 
        Log.e("dengjian", "静态缓存区域 name2:" + name2); 
        Log.e("dengjian", "静态缓存区域 name3:" + name3); 
        Log.e("dengjian", "静态缓存区域 name4:" + name4); 
        Log.e("dengjian", "静态缓存区域 name5:" + name5); 
        Log.e("dengjian", "静态缓存区域 name6:" + name6);
        
        staticCache("李小龙");
        Log.e("dengjian", "静态缓存区域 name1:" + name1); 
        Log.e("dengjian", "静态缓存区域 name2:" + name2); 
        Log.e("dengjian", "静态缓存区域 name3:" + name3); 
        Log.e("dengjian", "静态缓存区域 name4:" + name4); 
        Log.e("dengjian", "静态缓存区域 name5:" + name5); 
        Log.e("dengjian", "静态缓存区域 name6:" + name6);
        
        staticCache("李连杰");
        Log.e("dengjian", "静态缓存区域 name1:" + name1); 
        Log.e("dengjian", "静态缓存区域 name2:" + name2); 
        Log.e("dengjian", "静态缓存区域 name3:" + name3); 
        Log.e("dengjian", "静态缓存区域 name4:" + name4); 
        Log.e("dengjian", "静态缓存区域 name5:" + name5); 
        Log.e("dengjian", "静态缓存区域 name6:" + name6);
        
        staticCache("李贵");
        Log.e("dengjian", "静态缓存区域 name1:" + name1); 
        Log.e("dengjian", "静态缓存区域 name2:" + name2); 
        Log.e("dengjian", "静态缓存区域 name3:" + name3); 
        Log.e("dengjian", "静态缓存区域 name4:" + name4); 
        Log.e("dengjian", "静态缓存区域 name5:" + name5); 
        Log.e("dengjian", "静态缓存区域 name6:" + name6);
        
        staticCache("李逵");
        Log.e("dengjian", "静态缓存区域 name1:" + name1); 
        Log.e("dengjian", "静态缓存区域 name2:" + name2); 
        Log.e("dengjian", "静态缓存区域 name3:" + name3); 
        Log.e("dengjian", "静态缓存区域 name4:" + name4); 
        Log.e("dengjian", "静态缓存区域 name5:" + name5); 
        Log.e("dengjian", "静态缓存区域 name6:" + name6);
        
        staticCache("李鬼");
        Log.e("dengjian", "静态缓存区域 name1:" + name1); 
        Log.e("dengjian", "静态缓存区域 name2:" + name2); 
        Log.e("dengjian", "静态缓存区域 name3:" + name3); 
        Log.e("dengjian", "静态缓存区域 name4:" + name4);
        Log.e("dengjian", "静态缓存区域 name5:" + name5);
        Log.e("dengjian", "静态缓存区域 name6:" + name6); 
    }
    
    @Override
    protected void onDestroy() { 
        super.onDestroy();
        clearStaticCache(); // 必须要清除静态缓存
    } 
}

native-lib.cpp:
// 普通局部缓存 弊端演示
extern "C"
JNIEXPORT void JNICALL
Java_com_dengjian_as_1jni_1project_MainActivity2_localCache(JNIEnv *env, jclass clazz, jstring name) {
    // 像 OpenCV WebRtc 等 大量使用局部缓存
    // name属性 赋值操作
    static jfieldID f_id = nullptr; // 局部缓存，这个方法会被多次调用，不需要反复的去获取  
    if (f_id == nullptr) {
        f_id = env->GetStaticFieldID(clazz, "name1", "Ljava/lang/String;"); 
    } else {
        LOGD("fieldID是空的啊"); 
    }
    env->SetStaticObjectField(clazz, f_id, name); 
}

// TODO ====================== 下面是全局静态缓存区域 
// 全局静态缓存，在构造函数中初始化的时候会去缓存 
static jfieldID f_name1_id = nullptr;
static jfieldID f_name2_id = nullptr;
static jfieldID f_name3_id = nullptr; 
static jfieldID f_name4_id = nullptr; 
static jfieldID f_name5_id = nullptr;
static jfieldID f_name6_id = nullptr;

// 1 先初始化静态缓存(类似于 在构造方法里面先初始化缓存)
extern "C"
JNIEXPORT void JNICALL Java_com_dengjian_as_1jni_1project_MainActivity2_initStaticCache(JNIEnv *env, jclass clazz) {
    // 初始化全局静态缓存
    f_name1_id = env->GetStaticFieldID(clazz, "name1", "Ljava/lang/String;");
    f_name2_id = env->GetStaticFieldID(clazz, "name2", "Ljava/lang/String;");
    f_name3_id = env->GetStaticFieldID(clazz, "name3", "Ljava/lang/String;");
    f_name4_id = env->GetStaticFieldID(clazz, "name4", "Ljava/lang/String;");
    f_name5_id = env->GetStaticFieldID(clazz, "name5", "Ljava/lang/String;");
    f_name6_id = env->GetStaticFieldID(clazz, "name6", "Ljava/lang/String;");
    // 省略 ...
}

// 2 然后再 执行时，不会重复的去获取 jfieldID
extern "C"
JNIEXPORT void JNICALL
Java_com_dengjian_as_1jni_1project_MainActivity2_staticCache(JNIEnv *env, jclass clazz, jstring name) {
    // 如果这个方法会反复的被调用，那么不会反复的去获取 jfieldID，因为是先初始化静态缓存，然后再执行此函 数的
    env->SetStaticObjectField(clazz, f_name1_id, name);
    env->SetStaticObjectField(clazz, f_name2_id, name); 
    env->SetStaticObjectField(clazz, f_name3_id, name); 
    env->SetStaticObjectField(clazz, f_name4_id, name); 
    env->SetStaticObjectField(clazz, f_name5_id, name); 
    env->SetStaticObjectField(clazz, f_name6_id, name);
}

// 3 最后要清除静态缓存
extern "C"
JNIEXPORT void JNICALL Java_com_dengjian_as_1jni_1project_MainActivity2_clearStaticCache(JNIEnv *env, jclass clazz) {
    f_name1_id = nullptr; 
    f_name2_id = nullptr; 
    f_name3_id = nullptr; 
    f_name4_id = nullptr; 
    f_name5_id = nullptr; 
    f_name6_id = nullptr; 
    LOGD("静态缓存清除完毕...");
}
```
* native异常捕获
```c++
// 03.native异常捕获 ======================
// 异常方式一: 【C++处理时异常】 扭转乾坤
extern "C"
JNIEXPORT void JNICALL Java_com_dengjian_as_1jni_1project_MainActivity3_exception(JNIEnv *env, jclass clazz) {
    // 假设现在想操作name999，没有name999就会在native层奔溃掉
    jfieldID f_id = env->GetStaticFieldID(clazz, "name999", "Ljava/lang/String;");
    
    // 共两种方式 之 方式一
    // 补救措施 ，name999 拿不到报错的话，那么我就拿 name1
    jthrowable throwable = env->ExceptionOccurred(); // 检测本次函数执行，到底有没有异常 
    if (throwable) {
        // 补救措施，先把异常清除
        LOGD("native层:检测到 有异常...");
        // 清除异常
        env->ExceptionClear();
        // 重新获取 name1 属性
        f_id = env->GetStaticFieldID(clazz, "name1", "Ljava/lang/String;");
    }
}

// 异常方式二:【C++处理时异常】 往Java层抛出异常
extern "C"
JNIEXPORT void JNICALL Java_com_dengjian_as_1jni_1project_MainActivity3_exception2(JNIEnv *env, jclass clazz) {
    // 假设现在想操作name999，没有name999就会在native层奔溃掉
    jfieldID f_id = env->GetStaticFieldID(clazz, "name999", "Ljava/lang/String;");
    // 共两种方式 之 方式二
    // 补救措施 ，name999 拿不到报错的话，想给 java 层抛一个异常
    jthrowable throwable = env->ExceptionOccurred(); // 检测本次函数执行，到底有没有异常 // 想给 java 层抛一个异常
    if (throwable) {
        // 清除异常
        env->ExceptionClear();
        // Throw 抛一个 java 的 Throwable 对象
        jclass no_such_clz = env->FindClass("java/lang/NoSuchFieldException"); 
        env->ThrowNew(no_such_clz,"NoSuchFieldException 实在是找不到 name999 啊，没办法，奔溃了!");
    } 
}

// 异常方式三:【Java处理时异常】 Java的方法抛出了异常，然后native去清除
// 注意:Java的异常 native层无法捕获
extern "C"
JNIEXPORT void JNICALL Java_com_dengjian_as_1jni_1project_MainActivity3_exception3(JNIEnv *env, jclass clazz) {
    jmethodID showMID = env->GetStaticMethodID(clazz, "show", "()V"); env->CallStaticVoidMethod(clazz, showMID);
    
    // 按道理来说，上面的这句话:env->CallStaticVoidMethod(clazz, showMID);，就已经奔溃了，但是事实是 否如此呢?
    LOGI("native层:>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>.1"); LOGI("native层:>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>.2"); LOGI("native层:>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>.3"); LOGI("native层:>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>.4"); LOGI("native层:>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>.5"); LOGI("native层:>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>.6");
    
    // 证明不是马上就奔溃了，而是预料了时间，给我们处理呀 
    if (env->ExceptionCheck()) {
        env->ExceptionDescribe();// 输出描述
        env->ExceptionClear();// 清除异常 
    }
}

// TODO JNI异常处理总结:============================================== 
/*
Native层出错了，没有办法再Java层去try的，处理的方式一般有两种: 第一种:补救，例如:name999获取检测到发生异常了，就再去获取name0 第二种:抛出，例如:name999获取检测到发生异常了，把此异常抛给Java层，让Java层去捕获异常
Java层出错了，Native层可以去 监测到 然后清除Java的异常，具体业务逻辑自己去处理哦 
*/
```
* 手写JNIEnv
```c++
#include <iostream>
#include <string>
using namespace std;
// 如果是C语言
typedef const struct JNINativeInterface * JNIEnv; // 定义一个结构体指针的别名
// 模拟一个结构体
struct JNINativeInterface{
    // 结构体的函数指针
    char*(*NewStringUTF)(JNIEnv*, char*); // 函数指针的定义，实现在库中，我们这里还看不到 // 省略 300多个 函数指针
    // ...
};

typedef char * jstring; // 简化了
typedef char * jobject; // 简化了

// 函数指针 对应的 函数实现 (这里只是简写，真正复杂的代码，就不去考虑了) jstring NewStringUTF(JNIEnv* env, char* c_str) {
    // 注意:在真正的源码中，这里需要写很多复杂代码来转换的(毕竟涉及到跨语言操作了C<-->Java)，这里我们 就简写了
    // c_str -> jstring
    return c_str;
}

// 模拟 我们的 JNI 函数，重点关注形参 JNIEnv * env
char* Java_j07_Demo02_getCStringPwd(JNIEnv * env, jobject jobject){
    // JNIEnv * 其实已经是一个二级指针了，所以 -> 调用的情况下必须是一级指针 *取值
    // C语言就是，就是二级指针，所以需要*取出一级指针才能使用->， ->代表操作一级指针 
    return (*env)->NewStringUTF(env, "9527");
}

// 下面是测试端 -- 其实就是 模拟 JNIEnv 内部执行的过程 
int main() {
    // 构建 JNIEnv* 对象
    struct JNINativeInterface nativeInterface;
    // 给结构方法指针进行赋值(实现)
    nativeInterface.NewStringUTF = NewStringUTF;
    // 传给 Java_j07_Demo02_getCStringPwd 的参数是 JNIEnv*
    JNIEnv env = &nativeInterface;// 一级指针
    JNIEnv * jniEnv = &env;// 二级指针
    
    // 把 jniEnv 对象传给 Java_j07_Demo02_getCStringPwd Java层
    char* jstring = Java_j07_Demo02_getCStringPwd(jniEnv, "com/dengjian/jni/MainActivity");
    
    // jstring 通过 JNIEnv桥梁 传给 java 层 (这个过程也省略了... 直接打印了) 
    printf("Java层就拿到了 C++ 给我们的 jstring = %s", jstring);
    
    return 0; 
}
```
* 手写Parcel的C++层与原理
    * 以OpenCV部分源码举例（Java层Mat.java ------------- Native层Mat.cpp）：
        * OpenCV的官网：https://opencv.org/opencv-3-2/
        * OpenCV在线源码：https://github.com/opencv/opencv
        * Java层：看 Mat.java：
        https://github.com/opencv/opencv/blob/master/modules/core/misc/java/src/java/core%2BMat.java
        ```
        package org.opencv.core;
        
        import java.nio.ByteBuffer;
        
        // C++: class Mat
        //javadoc: Mat
        public class Mat {
        
            public final long nativeObj;  // Mat.cpp对象实例的首地址
        
            // javadoc: Mat::Mat()
            public Mat() {
        
                // C++创建一个对象，返回是long类型值（C++对象的指针地址）
                nativeObj = n_Mat();
            }
        
            ... 
        
            // C++: Mat::Mat()
            private static native long n_Mat();
        
            // C++: Mat::Mat(int rows, int cols, int type)
            private static native long n_Mat(int rows, int cols, int type);
        }
        ```
        * C++层：看Mat.cpp:
        在Github中搜索：Java_org_opencv_core_Mat_n_1Mat__DDI
        https://github.com/opencv/opencv/blob/fc1a15626226609babd128e043cf7c4e32f567ca/modules/java/generator/src/cpp/Mat.cpp
        ```
        // 函数的声明
        JNIEXPORT jlong JNICALL Java_org_opencv_core_Mat_n_1Mat__DDI
          (JNIEnv* env, jclass, jdouble size_width, jdouble size_height, jint type);
        
        // 函数的实现
        JNIEXPORT jlong JNICALL Java_org_opencv_core_Mat_n_1Mat__DDI
          (JNIEnv* env, jclass, jdouble size_width, jdouble size_height, jint type)
        {
            static const char method_name[] = "Mat::n_1Mat__DDI()";
            try {
                LOGD("%s", method_name);
                Size size((int)size_width, (int)size_height);
        
                // new Mat.cpp 实例 强行 转换
                //【重点这一句，new Mat(); C++对象，把此对象的首地址返回给Java==nativeObj】
                return (jlong) new Mat( size, type ); 
        
            } catch(const std::exception &e) {
                throwJavaException(env, &e, method_name);
            } catch (...) {
                throwJavaException(env, 0, method_name);
            }
        
            return 0;
        }
        ```
        * 总结：Java层拿到nativeObj就是 Native层C++对象的指针，也就是对象的首地址，下次想去使用Native层的功能时，就可以使用首地址寻找C++对象，并让他干活即可。
    * Parcel源码分析
```
//  mNativePtr == 拿到了 Parcel.cpp 对象指针地址

package android.os;

public final class Parcel {
    /**
     * 用于初始化 Parcel对象的函数
     * Retrieve a new Parcel object from the pool.
     */
    public static Parcel obtain() {
        final Parcel[] pool = sOwnedPool;
        synchronized (pool) {
            Parcel p;
            for (int i=0; i<POOL_SIZE; i++) {
                p = pool[i];
                if (p != null) {
                    pool[i] = null;
                    if (DEBUG_RECYCLE) {
                        p.mStack = new RuntimeException();
                    }
                    p.mReadWriteHelper = ReadWriteHelper.DEFAULT;
                    return p;
                }
            }
        }
        // 注意：这里的 new Parcel(0); 操作，会进入下面代码
        return new Parcel(0);
    }
}

private Parcel(long nativePtr) {
    if (DEBUG_RECYCLE) {
        mStack = new RuntimeException();
    }
    //Log.i(TAG, "Initializing obj=0x" + Integer.toHexString(obj), mStack

    // 注意：这里的 init函数，会继续你下面代码
    init(nativePtr);
}

private void init(long nativePtr) { // mNativePtr == Parcel.cpp 对象指针地址
    if (nativePtr != 0) {
        mNativePtr = nativePtr;
        mOwnsNativeParcelObject = false;
    } else {
        mNativePtr = nativeCreate(); // 注意：这里的nativeCreate();会返回C++的头指针内存地址
        mOwnsNativeParcelObject = true;
    }
}

private static native long nativeCreate(); // 注意：从这里进入Native层代码

// 动态注册
static const JNINativeMethod gParcelMethods[] = {
    // @CriticalNative
    {"nativeDataSize",            "(J)I", (void*)android_os_Parcel_dataSize},
    // @CriticalNative
    {"nativeDataAvail",           "(J)I", (void*)android_os_Parcel_dataAvail},
    // @CriticalNative
    {"nativeDataPosition",        "(J)I", (void*)android_os_Parcel_dataPosition},
    // @CriticalNative
    {"nativeDataCapacity",        "(J)I", (void*)android_os_Parcel_dataCapacity},
    // @FastNative
    {"nativeSetDataSize",         "(JI)J", (void*)android_os_Parcel_setDataSize},
    // @CriticalNative
    {"nativeSetDataPosition",     "(JI)V", (void*)android_os_Parcel_setDataPosition},
    // @FastNative
    {"nativeSetDataCapacity",     "(JI)V", (void*)android_os_Parcel_setDataCapacity},

    // @CriticalNative
    {"nativePushAllowFds",        "(JZ)Z", (void*)android_os_Parcel_pushAllowFds},
    // @CriticalNative
    {"nativeRestoreAllowFds",     "(JZ)V", (void*)android_os_Parcel_restoreAllowFds},

    {"nativeWriteByteArray",      "(J[BII)V", (void*)android_os_Parcel_writeByteArray},
    {"nativeWriteBlob",           "(J[BII)V", (void*)android_os_Parcel_writeBlob},
    // @FastNative
    {"nativeWriteInt",            "(JI)V", (void*)android_os_Parcel_writeInt},
    // @FastNative
    {"nativeWriteLong",           "(JJ)V", (void*)android_os_Parcel_writeLong},
    // @FastNative
    {"nativeWriteFloat",          "(JF)V", (void*)android_os_Parcel_writeFloat},
    // @FastNative
    {"nativeWriteDouble",         "(JD)V", (void*)android_os_Parcel_writeDouble},
    {"nativeWriteString",         "(JLjava/lang/String;)V", (void*)android_os_Parcel_writeString},
    {"nativeWriteStrongBinder",   "(JLandroid/os/IBinder;)V", (void*)android_os_Parcel_writeStrongBinder},
    {"nativeWriteFileDescriptor", "(JLjava/io/FileDescriptor;)J", (void*)android_os_Parcel_writeFileDescriptor},

    {"nativeCreateByteArray",     "(J)[B", (void*)android_os_Parcel_createByteArray},
    {"nativeReadByteArray",       "(J[BI)Z", (void*)android_os_Parcel_readByteArray},
    {"nativeReadBlob",            "(J)[B", (void*)android_os_Parcel_readBlob},
    // @CriticalNative
    {"nativeReadInt",             "(J)I", (void*)android_os_Parcel_readInt},
    // @CriticalNative
    {"nativeReadLong",            "(J)J", (void*)android_os_Parcel_readLong},
    // @CriticalNative
    {"nativeReadFloat",           "(J)F", (void*)android_os_Parcel_readFloat},
    // @CriticalNative
    {"nativeReadDouble",          "(J)D", (void*)android_os_Parcel_readDouble},
    {"nativeReadString",          "(J)Ljava/lang/String;", (void*)android_os_Parcel_readString},
    {"nativeReadStrongBinder",    "(J)Landroid/os/IBinder;", (void*)android_os_Parcel_readStrongBinder},
    {"nativeReadFileDescriptor",  "(J)Ljava/io/FileDescriptor;", (void*)android_os_Parcel_readFileDescriptor},

    {"openFileDescriptor",        "(Ljava/lang/String;I)Ljava/io/FileDescriptor;", (void*)android_os_Parcel_openFileDescriptor},
    {"dupFileDescriptor",         "(Ljava/io/FileDescriptor;)Ljava/io/FileDescriptor;", (void*)android_os_Parcel_dupFileDescriptor},
    {"closeFileDescriptor",       "(Ljava/io/FileDescriptor;)V", (void*)android_os_Parcel_closeFileDescriptor},

    // 在这里就找到了，【nativeCreate】
    {"nativeCreate",              "()J", (void*)android_os_Parcel_create},
    {"nativeFreeBuffer",          "(J)J", (void*)android_os_Parcel_freeBuffer},
    {"nativeDestroy",             "(J)V", (void*)android_os_Parcel_destroy},

    {"nativeMarshall",            "(J)[B", (void*)android_os_Parcel_marshall},
    {"nativeUnmarshall",          "(J[BII)J", (void*)android_os_Parcel_unmarshall},
    {"nativeCompareData",         "(JJ)I", (void*)android_os_Parcel_compareData},
    {"nativeAppendFrom",          "(JJII)J", (void*)android_os_Parcel_appendFrom},
    // @CriticalNative
    {"nativeHasFileDescriptors",  "(J)Z", (void*)android_os_Parcel_hasFileDescriptors},
    {"nativeWriteInterfaceToken", "(JLjava/lang/String;)V", (void*)android_os_Parcel_writeInterfaceToken},
    {"nativeEnforceInterface",    "(JLjava/lang/String;)V", (void*)android_os_Parcel_enforceInterface},

    {"getGlobalAllocSize",        "()J", (void*)android_os_Parcel_getGlobalAllocSize},
    {"getGlobalAllocCount",       "()J", (void*)android_os_Parcel_getGlobalAllocCount},

    // @CriticalNative
    {"nativeGetBlobAshmemSize",       "(J)J", (void*)android_os_Parcel_getBlobAshmemSize},
};

// 再接着搜索，(void*) **android_os_Parcel_create** 函数即可
static jlong android_os_Parcel_create(JNIEnv* env, jclass clazz)
{
    // 实例化C++ 对象，此对象 就是一个 内存地址指针
    Parcel* parcel = new Parcel();

    // 返回：内存地址指针，并且用reinterpret_cast 转换jlong类型，这代码一看就知道比OpenCV写得好
    return reinterpret_cast<jlong>(parcel);
}

// writeInt源码分析
/**
 * Write an integer value into the parcel at the current dataPosition(),
 * growing dataCapacity() if needed.
 */
public final void writeInt(int val) {

    // 注意：这里会调用native层，下面就会分析这个代码
    nativeWriteInt(mNativePtr, val);
}

@FastNative // 思考：行参一的作用是可以通过此内存地址去寻找native层的C++对象
private static native void nativeWriteInt(long nativePtr, int val); // 下面进入native

{"nativeWriteByteArray",      "(J[BII)V", (void*)android_os_Parcel_writeByteArray},
{"nativeWriteBlob",           "(J[BII)V", (void*)android_os_Parcel_writeBlob},
// @FastNative

// 还是采用动态注册的方式，搜索函数实现 android_os_Parcel_writeInt 即可
{"nativeWriteInt",            "(JI)V", (void*)android_os_Parcel_writeInt},

// @FastNative
{"nativeWriteLong",           "(JJ)V", (void*)android_os_Parcel_writeLong},
// @FastNative
{"nativeWriteFloat",          "(JF)V", (void*)android_os_Parcel_writeFloat},

static void android_os_Parcel_writeInt(JNIEnv* env, jclass clazz, jlong nativePtr, jint val) {

    // 通过在Java层保存的，C++对象首地址，来查找到C++对象 Parcel* parcel
    Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
    if (parcel != NULL) {
        // 把内容写入进去，这里是调用到哪里去？ 看下面代码...
        const status_t err = parcel->writeInt32(val); 
        if (err != NO_ERROR) {
            // 抛出异常
            signalExceptionForError(env, clazz, err);
        }
    }
}

status_t Parcel::writeInt32(int32_t val)
{
    return writeAligned(val);
}

template<class T> // 相当于Java的泛型
status_t Parcel::writeAligned(T val) {
    COMPILE_TIME_ASSERT_FUNCTION_SCOPE(PAD_SIZE_UNSAFE(sizeof(T)) == sizeof(T));

    // mDataPos：内存首地址的当前挪动位置
    // mDataCapacity：共享内存的总大小
    // mData：共享内存的首地址

    if ((mDataPos+sizeof(val)) <= mDataCapacity) {

        // 就相当于，指针++ 挪动好位置 然后存放刚刚挪动的位置
        // * mData共享内存的首地址 = val赋值给左边;
        restart_write: *reinterpret_cast<T*>(mData+mDataPos) = val;

        // 下面分析 finsihWrite函数
        return finishWrite(sizeof(val));
    }

    status_t err = growData(sizeof(val));
    if (err == NO_ERROR) goto restart_write;
    return err;
}

status_t Parcel::finishWrite(size_t len)
{
    if (len > INT32_MAX) {
        // don't accept size_t values which may have come from an
        // inadvertent conversion from a negative int.
        return BAD_VALUE;
    }

    //printf("Finish write of %d\n", len);
    mDataPos += len; // 【上面函数存值后，把指针位置进行挪动一次，方便后续的值 存放】

    ALOGV("finishWrite Setting data pos of %p to %zu", this, mDataPos);
    if (mDataPos > mDataSize) {
        mDataSize = mDataPos;
        ALOGV("finishWrite Setting data size of %p to %zu", this, mDataSize);
    }
    //printf("New pos=%d, size=%d\n", mDataPos, mDataSize);
    return NO_ERROR;
}
```