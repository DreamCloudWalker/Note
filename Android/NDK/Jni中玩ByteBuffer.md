### 相比传byte[]

如果jni函数需要返回java层一个byte[]，jni中需要频繁jbyteArray result =  \env->NewByteArray(size); 并且env->SetByteArrayRegion(result, 0, size, (jbyte *) data); 对性能会有影响。



这种情况下，可以从java层创建出ByteBuffer，当作jobject传到JNI , 再通过env->GetDirectBufferAddress转成指针对象，让C++的库用这个指针去填充这段内存。达到复用的逻辑。

注意ByteBuffer要在***Java层开辟***，不然不会被GC维护。

示例：

```c++
#include <chrono>

static jboolean libxxx_process(JNIEnv *env, jobject thiz, 
                               jbyteArray buffer,		// 源数据，比如相机nv21数据
                               jobject retBuffer) {	// retBuffer 是在Java开辟的ByteBuffer
    jboolean jCopy = false;
    unsigned char *pAddress = (unsigned char *) env->GetByteArrayElements(buffer, &jCopy);
    if (nullptr == pAddress) {
        return false;
    }

    if (nullptr == retBuffer) { // manage ByteBuffer at Java
        return false;
    }
  
    jclass bufClass = env->GetObjectClass(retBuffer);
    jmethodID clearID = env->GetMethodID(bufClass, "clear", "()Ljava/nio/Buffer;");
    env->CallObjectMethod(retBuffer, clearID);

    unsigned char *pDstAddress = reinterpret_cast<unsigned char *> (env->GetDirectBufferAddress(
      retBuffer));

    auto tick = std::chrono::high_resolution_clock::now();
    ret = xxxLib->process(pAddress, pDstAddress);
    auto now = std::chrono::high_resolution_clock::now();
}
```

