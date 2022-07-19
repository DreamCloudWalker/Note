

## eglMakeCurrent

记得这个调用吗？方法原型是：

```
EGLBoolean eglMakeCurrent(
    EGLDisplay display,
    EGLSurface draw,
    EGLSurface read,
    EGLContext context
);
```

为了弄懂这个方法，我们需要搞清楚`Display`、`Surface`、`Context`这几个概念。



## Display

在使用EGL 的过程中，会发现EGL 相关的调用都经常需要我们传入一个类型为`Display`的参数。

**顾名思义，`Display`是一个连接，用于连接设备上的底层窗口系统。**

所以在调用EGL 方法之前，需要先创建、初始化这个`Display`连接。步骤如下：

```java
EGLint majorVersion;
EGLint minorVersion;
EGLDisplay display;
display = eglGetDisplay(EGL_DEFAULT_DISPLAY);
if (display == EGL_NO_DISPLAY) {
    // 无法打开到底层窗口系统的连接
}
if (!eglInitialize(display, &majorVersion, &minorVersion)) {
    // 无法初始化EGL
}
```

通常情况下传入`EGL_DEFAULT_DISPLAY`作为`eglGetDisplay`的参数就可以了，EGL 会自动返回默认的`Display`。



## Context

`Context`不是什么神秘的东西，**它仅仅是一个容器**，里面放着两个东西：

- **内部状态信息**(View port, depth range, clear color, textures, VBO, FBO, ...)
- **调用缓存**，保存了在这个`Context`下发起的GL调用指令。(OpenGL 调用是异步的)

总的来说，`Context`是设计来存储渲染相关的输入数据。



## Surface

**对应的`Surface`则是设计来存储渲染相关的输出数据。**

`Surface`实际上是**一个对底层窗口对象的拓展**、或是一个有着额外辅助缓冲的像素映射(pixmap)。这些辅助缓存包括颜色缓存(color buffer)、深度缓冲(depth buffer)、模板缓冲(stencil buffer)。



## 再回到 eglMakeCurrent

那么eglMakeCurrent到底做了什么？

当发起 GL 调用指令(如：glDrawElements)的时候，这个调用会影响到哪个Context和Surface呢？**答案就在****`eglMakeCurrent`****里**。

```
EGLBoolean eglMakeCurrent(
    EGLDisplay display,
    EGLSurface draw,
    EGLSurface read,
    EGLContext context
);
```

**`eglMakeCurrent`把`context`绑定到当前的渲染线程以及`draw`和`read`指定的Surface。**

draw 用于除数据回读(glReadPixels、glCopyTexImage2D和glCopyTexSubImage2D)之外的所有GL 操作。

回读操作作用于read指定的Surface上的帧缓冲(frame buffer)。

因此，当我们在线程T上调用GL 指令，OpenGL ES 会查询T线程绑定是哪个Context C，进而查询是哪个Surface draw和哪个Surface read绑定到了这个Context C上。



## 多个Context

某些情况下，我们想创建、使用多个Context，对于这种情况，需要注意以下几个情况：

- **不能在2个线程里绑定同一个`Context`**。
- **不能在2个不同的线程里，绑定相同的`Surface`到2个不同的`Context`上**。
- 在2个不同的线程里，绑定2个不同`Surface`到2个`Context`上，取决于使用的GPU的具体实现，**可能成功，也可能失败**。



## 共享Context

共享Context这种方式在加载阶段很有用。**由于上传数据到GPU(尤其是纹理数据(textures))这类操作很重，如果想要维持帧率稳定，应该在另一个线程进行上传**。

然而，出于上述3种情况的限制，必须在第一个Context之外，创建第二个Context，这个Context将使用第一个Context使用的内部状态信息。这两个Context即共享Context。

需要注意的是：**这两个Context共享的只是内部状态信息，它们两个并不共享调用缓存(每个Context各自拥有一个调用缓存)。**

创建第二个Context的方法：

```
EGLContext eglCreateContext(
    EGLDisplay display,
    EGLConfig config,
    EGLContext share_context,
    EGLint const * attrib_list);
```

第三个参数`share_context`是最重要的，它就是第一个Context。

在第二个线程，不进行任何的绘制，只进行上传数据到GPU 的操作。所以，给第二个Context 的Surface 应该是一个像素缓冲(pixel buffer)Surface。

```
EGLSurface eglCreatePbufferSurface(
    EGLDisplay display,
    EGLConfig config,
    EGLint const * attrib_list);
```



## eglSwapBuffers

```
EGLBoolean eglSwapBuffers(
    EGLDisplay display,
    EGLSurface surface);
```

第一次看到这个调用的时候，我以为它的作用是对`display`和`surface`进行交换:) 槑

实际上，这里需要重点注意的是`surface`。如果这里的`surface
是一个像素缓冲(pixel buffer)Surface，那什么都不会发生，调用将正确的返回，不报任何错误。

但如果`surface`是一个双重缓冲surface(大多数情况)，这个方法将会交换`surface`内部的前端缓冲(front-buffer)和后端缓冲(back-surface)。

后端缓冲用于存储渲染结果，前端缓冲则用于底层窗口系统，底层窗口系统将缓冲中的颜色信息显示到设备上。



## glFlush 和 glFinish

OpenGL ES 驱动和GPU以并行/异步的机制运行。发起GL 调用时，为了得到最好的性能，驱动会尝试尽快地把调用指令发送给GPU。

**但GPU 并不会马上执行这些指令，这些指令只是添加到了GPU的指令队列里等待GPU 执行。**

如果在短时间内发送大量的GL 指令给GPU，GPU的指令队列可能满了，以至于驱动需要把这些指令保存在Context的调用缓存里(即上文里提到的Context内的调用缓存)。

那么问题来了，这些等待中的指令何时会发送给GPU呢？

**通常来说，大多数`OpenGL ES`驱动的实现可能会在发起新的(下一个)GL 指令发起的时候发送这些指令。如果想要主动执行这个操作，那就调用`glFlush`。**

这个操作将会阻塞当前线程，直到所有的指令都发送给了`GPU`。`glFinish`这个命令更加强大，它会阻塞当前线程，直到所有的指令都发送给了GPU，并执行完毕。需要注意的是，应用程序的性能会因此下降。

**`glFlush`和`glFinish`被称为显式同步操作。某些情况下也会发生隐式同步操作。调用`eglSwapBuffers`时，就可能发生这种情况。**

由于这个操作是由驱动直接执行的，此时GPU 可能把所有待执行的`glDraw*`绘制指令，作用在一个不符合预期的surface 缓冲上(如果之前前端缓冲和后端缓冲已经交换过了)。

**为了防止这种情形，在交换缓冲前，驱动必须阻塞当前线程，等待所有的影响当前surface的`glDraw\*`指令执行完毕。**

当然，使用双重缓冲的surfaces时，不需要主动调用`glFlush`或`glFinish`：因为`eglSwapBuffers`进行了**隐式**同步操作。

但在使用单缓冲surfaces(如上文提到的第二个线程里)的情况，需要及时调用`glFlush`，例如：在线程退出前，必须调用`glFlush`，否则，GL 指令可能从未发送到GPU。



## End

原文链接：Let’s talk about eglMakeCurrent, eglSwapBuffers, glFlush, glFinish

https://katatunix.wordpress.com/2014/09/17/lets-talk-about-eglmakecurrent-eglswapbuffers-glflush-glfinish/



> *原文链接: https://juejin.cn/post/6844903472085598216*