摘自：[https://zhuanlan.zhihu.com/p/24762605](https://links.jianshu.com/go?to=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F24762605)

## 简介

 在iOS里，我们经常能看到 CVPixelBufferRef 这个类型，在Camera 采集返回的数据里得到一个CMSampleBufferRef，而每个CMSampleBufferRef里则包含一个 CVPixelBufferRef，在视频硬解码的返回数据里也是一个 CVPixelBufferRef。
 顾名思义，CVPixelBufferRef 是一种像素图片类型，由于CV开头，所以它是属于 CoreVideo 模块的。

iOS喜欢在对象命名前面用缩写表示它属于的模块，比如 CF 代表 CoreFoundation，CG代表 CoreGraphic，CM代表 CoreMedia。既然属于 CoreVideo那么它就和视频处理相关了。

它是一个C对象，而不是Objective C对象，所以它不是一个类，而是一个类似Handle的东西。从代码头文件的定义来看

CVPixelBufferRef 就是用 CVBufferRef typedef而来的，而CVBufferRef 本质上就是一个void *，至于这个void *具体指向什么数据只有系统才知道了。

所以我们看到 所有对 CVPixelBufferRef 进行操作的函数都是纯 C 函数，这很符合iOS CoreXXXX系列 API 的风格。

比如 CVPixelBufferGetWidth， CVPixelBufferGetBytesPerRow

通过API可以看出来，CVPixelBufferRef里包含很多图片相关属性，比较重要的有 width，height，PixelFormatType等。

由于可以有不同的PixelFormatType，说明他可以支持多种位图格式，除了常见的 RGB32以外，还可以支持比如 kCVPixelFormatType_420YpCbCr8BiPlanarFullRange，这种YUV多平面的数据格式，这个类型里 BiPlanar 表示双平面，说明它是一个 NV12的YUV，包含一个Y平面和一个UV平面。通过CVPixelBufferGetBaseAddressOfPlane可以得到每个平面的数据指针。在得到 Address之前需要调用CVPixelBufferLockBaseAddress，这说明CVPixelBufferRef的内部存储不仅是内存也可能是其它外部存储，比如现存，所以在访问前要lock下来实现地址映射，同时lock也保证了没有读写冲突。

由于是C对象，它是不受 ARC 管理的，就是说要开发者自己来管理引用计数，控制对象的生命周期，可以用CVPixelBufferRetain，CVPixelBufferRelease函数用来加减引用计数，其实和CFRetain和CFRelease是等效的，所以可以用 CFGetRetainCount来查看当前引用计数。

如果要显示 CVPixelBufferRef 里的内容，通常有以下几个思路。



## 应用

1. 把 CVPixelBufferRef 转换成 UIImage，就可以直接赋值给UIImageView的image属性，显示在UIImageView上，示例代码：

```objective-c
(UIImage)uiImageFromPixelBuffer:(CVPixelBufferRef)p {
  CIImage ciImage = [CIImage imageWithCVPixelBuffer:p];

  CIContext* context = [CIContext contextWithOptions:@{kCIContextUseSoftwareRenderer : @(YES)}];

  CGRect rect = CGRectMake(0, 0, CVPixelBufferGetWidth(p), CVPixelBufferGetHeight(p));
  CGImageRef videoImage = [context createCGImage:ciImage fromRect:rect];

  UIImage* image = [UIImage imageWithCGImage:videoImage];
  CGImageRelease(videoImage);

  return image;
}
```

从代码可以看出来，这个转换有点复杂，中间经历了多个步骤，所以性能是很差的，只适合偶尔转换一张图片，用于调试截图等，用于显示视频肯定是不行的。



2. 另一个思路是用OpenGL来渲染，CVPixelBufferRef是可以转换成一个 openGL texture的，方法如下：

```objective-c
CVOpenGLESTextureRef pixelBufferTexture; CVOpenGLESTextureCacheCreateTextureFromImage(kCFAllocatorDefault,
_textureCache,
pixelBuffer,
NULL,
GL_TEXTURE_2D,
GL_RGBA,
width,
height,
GL_BGRA,
GL_UNSIGNED_BYTE,
0,
&pixelBufferTexture);
```

其中，_textureCache 代表一个 Texture缓存，每次生产的Texture都是从缓存获取的，这样可以省掉反复创建Texture的开销，_textureCache要实现创建好，创建方法如下

CVOpenGLESTextureCacheCreate(kCFAllocatorDefault, NULL, _context, NULL, &_textureCache);
 其中 _context 是 openGL的 context，在iOS里就是 EAGLContext *

pixelBufferTexture还不是openGL的Texture，调用CVOpenGLESTextureGetName才能获得在openGL可以使用的Texture ID。

当获得了 Texture ID后就可以用openGL来绘制了，这里推荐用 GLKView 来做绘制

```c++
glUseProgram(_shaderProgram);
    
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, textureId);
glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

glDrawArrays(GL_TRIANGLE_FAN, 0, 4);
```

当然这不是全部代码，完整的绘制openGL代码还有很多，openGL是著名的啰嗦冗长，还有openGL Context创建 shader编译 DataBuffer加载等。

本质上这段代码是为了把 Texture的内容绘制到 openGL的frame buffer里，然后再把frame buffer贴到 CAEAGLayer。

这个从CVPixelBufferRef获取的texture，和原来的CVPixelBufferRef 对象共享同一个存储，就是说如果改变了Texture的内容，那么CVPixelBufferRef的内容也会改变。利用这一点我们就可可以用openGL的绘制方法向CVPixelBufferRef对象输出内容了。比如可以给CVPixelBufferRef的内容加图形特效打水印等。



3. 除了从系统API里获得CVPixelBufferRef外，我们也可以自己创建CVPixelBufferRef

```objective-c
+(CVPixelBufferRef)createPixelBufferWithSize:(CGSize)size {
  const void *keys[] = {
    kCVPixelBufferOpenGLESCompatibilityKey,
    kCVPixelBufferIOSurfacePropertiesKey,
  };
  const void *values[] = {
    (__bridge const void *)([NSNumber numberWithBool:YES]),
    (__bridge const void *)([NSDictionary dictionary])
  };

  OSType bufferPixelFormat = kCVPixelFormatType_32BGRA;
  CFDictionaryRef optionsDictionary = CFDictionaryCreate(NULL, keys, values, 2, NULL, NULL);

  CVPixelBufferRef pixelBuffer = NULL;
  CVPixelBufferCreate(kCFAllocatorDefault,
                      size.width,
                      size.height,
                      bufferPixelFormat,
                      optionsDictionary,
                      &pixelBuffer);

  CFRelease(optionsDictionary);

  return pixelBuffer;
}
```

创建一个 BGRA 格式的PixelBuffer，注意kCVPixelBufferOpenGLESCompatibilityKey和kCVPixelBufferIOSurfacePropertiesKey这两个属性，这是为了实现和openGL兼容，另外有些地方要求CVPixelBufferRef必须是 IO Surface。
 CVPixelBufferRef是iOS视频采集处理编码流程的重要中间数据媒介和纽带，理解CVPixelBufferRef有助于写出高性能可靠的视频处理。

要进一步理解CVPixelBufferRef还需要学习 YUV，color range，openGL等知识。



