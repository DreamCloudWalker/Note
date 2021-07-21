### 简介

EGLImage代表一种由EGL客户API（如OpenGL，OpenVG）创建的共享资源类型。它的本意是共享2D图像数据，但是并没有明确限定共享数据的格式以及共享的目的，所以理论上来讲，应用程序以及相关的客户API可以基于任意的目的创建任意类型的共享数据。

关于EGLImage的一种使用情景就是通过它来创建一个2D纹理。相关函数原型声明如下：

EGLAPI EGLImageKHR EGLAPIENTRYeglCreateImageKHR (EGLDisplay dpy, EGLContext ctx, EGLenum target,EGLClientBuffer buffer, const EGLint *attrib_list);

这里我们主要关注下target，buffer参数。target决定了创建EGLImage的方式，例如在Android系统中专门定义了一个称为EGL_NATIVE_BUFFER_ANDROID的Target，支持通过ANativeWindowBuffer创建EGLImage对象，而Buffer则对应创建EGLImage对象时使用数据。

使用方式如下：

假设我们通过Gralloc模块创建了一个ANativeWindowBuffer对象srcBuf，它将被当作EGLClientBuffer类型的参数传递给上述函数，调用形式如下：

```
EGLImageKHR eglSrcImage = eglCreateImageKHR(
                        eglDisplay, 
                        EGL_NO_CONTEXT,
                        EGL_NATIVE_BUFFER_ANDROID,
                        (EGLClientBuffer) &sSrcBuffer,
                        0);
```

创建了一个EGLImage对象后，我们可以通过它创建一个2D纹理对象，示例如下：
```
glGenTextures(1,&texID);
glBindTexture(GL_TEXTURE_2D,texID);
glEGLImageTargetTexture2DOES(GL_TEXTURE_2D,eglSrcImage); // 相当于一般情况下的glTexImage2D的效果 
```
如果EGLClientBuffer的数据是YUV格式的，还可以使用纹理Target为：GL_TEXTURE_EXTERNAL_OES， 该Target也是主要用于从EGLImage中产生纹理的情景，并且它与GL_TEXTURE_2D不同，它只能用于此种情景，也就是说***glTexImage***( GL_TEXTURE_EXTERNAL_OES,…)是非法的。

这种创建纹理的方式使得显示一些OpenGL ES本身并不支持的图像格式的数据成为可能，因为它将格式支持相关的工作移到到了EGLImage创建函数当中。

### 实例
对于api >= 26(Android O)的时候，可以考虑用hardbuffer来提升glReadPixel的效率。比如：
```
typedef struct _HardBuffer{
    AHardwareBuffer_Desc desc;
    AHardwareBuffer *graphicBuffer;
    EGLImageKHR imageEGL;
    EGLDisplay display;
}HardBuffer;

...

AHardwareBuffer* graphicBuf;
if (AHardwareBuffer_allocate(&usage, &graphicBuf) != 0) {
    LOGD("AHardwareBuffer_allocate failed!");
    return JNI_FALSE;
}

// for stride, see below
AHardwareBuffer_describe(graphicBuf, &retriever->desc);

// get the native buffer
EGLClientBuffer clientBuf = eglGetNativeClientBufferANDROID(graphicBuf);

// obtaining the EGL display
EGLDisplay disp = eglGetDisplay(EGL_DEFAULT_DISPLAY);

// specifying the image attributes
EGLint eglImageAttributes[] = {EGL_IMAGE_PRESERVED_KHR, EGL_TRUE, EGL_NONE};

// creating an EGL image
retriever ->imageEGL = eglCreateImageKHR(disp, EGL_NO_CONTEXT, EGL_NATIVE_BUFFER_ANDROID, clientBuf, eglImageAttributes);

/**
 * @note this part should be earlies than any draw or framebuffer options.
 * @note refer to answer of @solidpixel at https://stackoverflow.com/questions/64447069/use-gleglimagetargettexture2does-to-replace-glreadpixels-on-android
 * @{
 */
retriever -> graphicBuffer = graphicBuf;
retriever -> display = disp;

...
// binding the OUTPUT texture
glBindTexture(GL_TEXTURE_2D, textureId);
// attaching an EGLImage to OUTPUT texture
glEGLImageTargetTexture2DOES(GL_TEXTURE_2D, retriever->imageEGL);

// readpixel,jni从外部传入一个ByteBuffer用来接收结果
void *pSourceAddress = env->GetDirectBufferAddress(byteBuf);
jlong dstSize;
jbyteArray byteArray = NULL;
if (pSourceAddress == NULL) {
    jclass byteBufClass = env->FindClass("java/nio/ByteBuffer");
    jmethodID arrayID =
            env->GetMethodID(byteBufClass, "array", "()[B");
    byteArray =
            (jbyteArray)env->CallObjectMethod(byteBuf, arrayID);
    if (byteArray == NULL) {
        return -1;
    }
    jboolean isCopy;
    pSourceAddress = env->GetByteArrayElements(byteArray, &isCopy);
    dstSize = env->GetArrayLength(byteArray);
} else {
    dstSize = env->GetDirectBufferCapacity(byteBuf);
}

void *readPtr;
uint8_t *writePtr = (uint8_t *)pSourceAddress + offset;
    
// locking the buffer
AHardwareBuffer_lock(retriever->graphicBuffer,
        AHARDWAREBUFFER_USAGE_CPU_READ_OFTEN,
        -1,
        nullptr,
        (void**) &readPtr); // worth checking return code


int width = retriever->desc.width;
int height = retriever->desc.height;
int stride = retriever->desc.stride;

uint8_t *pRead = (uint8_t*)readPtr;

// loop over texture rows
for (int row = 0; row < height; row++) {
    // copying, 4 = 4 channels RGBA because of the format above
    memcpy(writePtr, pRead, width * 4);

    // adding stride * 4 to read pointer
    pRead +=  stride * 4;

    // adding width * 4 to write pointer
    writePtr +=  width * 4;
}
// NOW data is in writePtr memory
// unlocking the buffer
AHardwareBuffer_unlock(retriever->graphicBuffer, nullptr); // worth checking return code

```