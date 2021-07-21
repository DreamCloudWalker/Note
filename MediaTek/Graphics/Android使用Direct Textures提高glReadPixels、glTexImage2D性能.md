本文档描述改善glReadPixels读取帧缓冲区数据在华为等使用Mali GPU的手机上速度慢的办法。因产品要求应用支持最低平台为Android 4.1，故无法通过Pixel Buffer Object（OpenGL ES 3.0接口，需Android 4.3）提高glReadPixels性能。那么，剩下就一种办法：使用Direct Textures（EGLImage），这是EGL拓展，适用于需要经常更新纹理数据的场合，比如逐帧更新。可用于OpenGL ES 1.0及2.0。

### 代码示例
Direct Textures用glEGLImageTargetTexture2DOES接口替代glReadPixels，它依赖于GraphicBuffer数据结构。算法描述如下：

* 指定宽高及像素格式初始化GraphicsBuffer
* 锁定
* 读写纹理数据
* 解锁

值得注意的是，一旦解锁，写入的数据将立即反馈在屏幕上。

初始化代码示例如下：
```
#include <GLES2/gl2.h>
#include <GLES2/gl2ext.h>
#include <EGL/egl.h>
#include <EGL/eglext.h>
#include <android/native_window.h>
#include <ui/GraphicBuffer.h>
#include <dlfcn.h>
// .......
GraphicBuffer* buffer = new GraphicBuffer(1024/*width*/, 1024/*height*/, 
                                          PIXEL_FORMAT_RGB_565,
                                          GraphicBuffer::USAGE_SW_WRITE_OFTEN |
                                          GraphicBuffer::USAGE_HW_TEXTURE);

unsigned char* bits = NULL;
buffer->lock(GraphicBuffer::USAGE_SW_WRITE_OFTEN, (void**)&bits);
// Write bitmap data into 'bits' here
buffer->unlock();

// Create the EGLImageKHR from the native buffer
EGLint eglImgAttrs[] = { EGL_IMAGE_PRESERVED_KHR, EGL_TRUE, EGL_NONE, EGL_NONE };
EGLImageKHR img = eglCreateImageKHR(eglGetDisplay(EGL_DEFAULT_DISPLAY), EGL_NO_CONTEXT,
                                    EGL_NATIVE_BUFFER_ANDROID,
                                    (EGLClientBuffer)buffer->getNativeBuffer(),
                                    eglImgAttrs);

// Create GL texture, bind to GL_TEXTURE_2D, etc.

// Attach the EGLImage to whatever texture is bound to GL_TEXTURE_2D
glEGLImageTargetTexture2DOES(GL_TEXTURE_2D, img);
```

1、使用glEGLImageTargetTexture2DOES替换glTexImage2D或glTexSubImage2D。
```
// glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, textureWidth, textureHeight, 0,GL_RGBA, GL_UNSIGNED_BYTE, NULL);
glEGLImageTargetTexture2DOES(GL_TEXTURE_2D, EGLImage);
```

2、使用glEGLImageTargetTexture2DOES替换glReadPixels。
```
// glReadPixels(0, 0, 
//              textureWidth, textureHeight, 
//              GL_RGBA, 
//              GL_UNSIGNED_BYTE, 
//              dataReadFromFramebuffer);
glEGLImageTargetTexture2DOES(GL_TEXTURE_2D, pEGLImage);
```
值得注意的是，由于glReadPixels及其等价函数默认读取后台帧缓冲区，故需要在eglSwapBuffers前调用这些函数。

根据android 下使用Direct Texture，Android 3.0等老版本因Android EGL库存在缺陷，故需手工加载Mali等GPU驱动。在3.0之后，此问题被修复，因而直接用eglGetProcAddress替代dlopen更为简单，完整示例代码如下所示。
```
const char* const driver_absolute_path = "/system/lib/egl/libEGL_mali.so";
// On Gingerbread you have to load symbols manually from Mali driver because
// Android EGL library has a bug.
// From  ICE CREAM SANDWICH you can freely use the eglGetProcAddress function.
// You might be able to get away with just eglGetProcAddress (no dlopen). 
// Try it,  else revert to the following code
void* dso = dlopen(driver_absolute_path, RTLD_LAZY);
if (dso != 0)
{
    LOGI("dlopen: SUCCEEDED");
    _eglCreateImageKHR = (PFNEGLCREATEIMAGEKHRPROC)dlsym(dso, "eglCreateImageKHR");
    _eglDestroyImageKHR = (PFNEGLDESTROYIMAGEKHRPROC) dlsym(dso,"eglDestroyImageKHR");
}
else
{
    LOGI("dlopen: FAILED! Loading functions in common way!");
    _eglCreateImageKHR = (PFNEGLCREATEIMAGEKHRPROC) eglGetProcAddress("eglCreateImageKHR");
    _eglDestroyImageKHR = (PFNEGLDESTROYIMAGEKHRPROC) eglGetProcAddress("eglDestroyImageKHR");
}
 
if(_eglCreateImageKHR == NULL)
{
    LOGE("Error: Failed to find eglCreateImageKHR at %s:%in", __FILE__, __LINE__);
    exit(1);
}
if(_eglDestroyImageKHR == NULL)
{
    LOGE("Error: Failed to find eglDestroyImageKHR at %s:%in", __FILE__, __LINE__);
    exit(1);
}
_glEGLImageTargetTexture2DOES = (PFNGLEGLIMAGETARGETTEXTURE2DOESPROC) eglGetProcAddress("glEGLImageTargetTexture2DOES");
if(_glEGLImageTargetTexture2DOES == NULL)
{
    LOGI("Error: Failed to find glEGLImageTargetTexture2DOES at %s:%in", __FILE__, __LINE__);
    return 0;
}
     
graphicBuffer = new GraphicBuffer(width, height,
            HAL_PIXEL_FORMAT_RGBA_8888,
            GraphicBuffer::USAGE_HW_TEXTURE |
            GraphicBuffer::USAGE_HW_2D |
            GRALLOC_USAGE_SW_READ_OFTEN |
            GRALLOC_USAGE_SW_WRITE_OFTEN);

status_t err = graphicBuffer->initCheck();
if (err != NO_ERROR)
{
    LOGI("Error: %sn", strerror(-err));
    return 0;
}

GGLSurface t;
graphicBuffer->lock(&t, GRALLOC_USAGE_SW_WRITE_OFTEN);
memset(t.data, 128, t.stride * t.height);
graphicBuffer->unlock();

// Retrieve andorid native buffer
android_native_buffer_t *anb = graphicBuffer->getNativeBuffer();
// create the new EGLImageKHR
const EGLint attrs[] =
{
    EGL_IMAGE_PRESERVED_KHR, 
    EGL_TRUE,
    EGL_NONE, 
    EGL_NONE
};

mEngine.mTexture.pEGLImage = _eglCreateImageKHR(eglGetCurrentDisplay(),
                                                mEngine.nContext, 
                                                EGL_NATIVE_BUFFER_ANDROID, 
                                                (EGLClientBuffer)anb, 
                                                attrs);
if(mEngine.mTexture.pEGLImage == EGL_NO_IMAGE_KHR)
{
    LOGI("Error: eglCreateImage() failed at %s:%in", __FILE__, __LINE__);
    return 0;
}
checkGlError("eglCreateImageKHR");
// Create Program等常规初始化操作
```

### 编译环境
由于Android NDK不暴露以上接口，意味着使用Direct Textures需要下载Android源码，编译并打包成动态库。接着，通过dlopen或eglGetProcAddress获取eglCreateImageKHR等接口的地址，再进行调用。

编译时包含头文件：
```
LOCAL_C_INCLUDES +=
    $(ANDROID_SRC_HOME)/frameworks/base/core/jni/android/graphics 
    $(ANDROID_SRC_HOME)/frameworks/base/include/
    $(ANDROID_SRC_HOME)/hardware/libhardware/include
    $(ANDROID_SRC_HOME)/system/core/include
    $(ANDROID_SRC_HOME)/frameworks/base/native/include/
    $(ANDROID_SRC_HOME)/frameworks/base/opengl/include/
```

链接选项：
```
LOCAL_LDLIBS := -llog -lGLESv2 -lEGL -landroid  -lui -landroid_runtime  -ljnigraphics
```

### 存在的问题
根据火狐（Mozilla Firefox）工程师2011年的博客：using direct textures on android，使用Direct Textures因需要调用dlopen接口，Adreno和Mali等GPU驱动不允许常规应用这么操作。
If you’ve ever used the Android NDK, it won’t be surprising that GraphicBuffer (or anything similar) doesn’t exist there. In order to use any of this in your app you’ll need to resort to dlopen hacks. It’s a pretty depressing situation. Google uses this all over the OS, but doesn’t seem to think that apps need a high performance API. But wait, it gets worse. Even after jumping through these hoops, some gralloc drivers don’t allow regular apps to play ball. So far, testing indicates that this is the case on Adreno and Mali GPUs. Thankfully, PowerVR and Tegra allow it, which covers a fair number of devices.

此时可尝试使用eglGetProcAddress替换dlopen接口。

### 参考
* using direct textures on android
* android 下使用Direct Texture
* EGL_KHR_image_base
* EGLImage - updating a texture without copying memory under Android 起始提供了一个预编译的Android运行环境的包装资源，然而被Mali GPU的合作伙伴要求撤消，故此链接参考意义不大。