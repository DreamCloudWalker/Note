## 一、简介

Vulkan API最常用的用法就是用来**绘制和显示图像（Presentation）**。

但是因为支持Vulkan运行的平台很多，有些以计算为目的应用也不需要把绘制的结果呈现给用户，所以显示相关的API并不是Vulkan核心API的一部分，而是通过扩展（extensions）的方式提供。



## 二、Swap Chains

每一个支持显示的平台都有自己的窗口系统Windowing System Integration (WSI)。

**Vulkan提供了Swap Chains 跟平台的窗口系统对接。**

Swap Chains**会向窗口系统申请一个或者多个可以用于显示的image对象**。我们可以将这些image对象作为我们的绘制目标，绘制完成后通过`vkQueuePresentKHR()`函数送显。

<img src=".asserts/image-20230419181823918.png" alt="image-20230419181823918" style="zoom:50%;" />

## 三、显示模式

Presentation（送显）功能依赖于平台提供支持，不同平台的特性有所差异。我们在通过Swap Chains 获取用来显示的image object之后，还需要确认当前平台支持的显示模式。

有些平台可能支持多种显示模式，我们需要根据自己的场景做出选择。

Vulkan API中定义了四种显示模式：

- IMMEDIATE
- FIFO
- FIFO RELAXED
- MAILBOX

### 3.1 IMMEDIATE

**IMMEDIATE**：送显之后立刻显示，替换当前正在显示的图像，不会关心vsync。

显示引擎中也没有等待队列保存图像。

这种方式不在消隐期间替换图像，很容易被用户看到撕裂现象。

<img src=".asserts/image-20230419181845868.png" alt="image-20230419181845868" style="zoom:50%;" />

### 3.2 FIFO 和 FIFO RELAXED

**FIFO**：**每个Vulkan API的实现都必须支持FIFO模式**。这种模式下，送显的图像会先被保存在一个先进先出的队列中，当vsync触发时，队列里的图像才会替换当前显示的图像，被替换的图像可以重新被应用程序申请绘制。

<img src=".asserts/image-20230419181912603.png" alt="image-20230419181912603" style="zoom:50%;" />

**FIFO RELAXED** **在FIFO上做了修改，当图像的显示时长超过一个vsync周期时，下一个图像不会再等待下一次vsync的到来**，而是立刻显示。如果这个操作在非消隐期间，用户可以看到撕裂现象。

### 3.3 MAILBOX

**MAILBOX**：该模式下，图像仅仅在消隐期间显示，所以用户看不到撕裂现象。在显示引擎内部，只使用单个元素的队列，当有新的图像被生成时，等待队列的图像将被替换。所以这种模式下，显示的图像永远是最新的。

<img src=".asserts/image-20230419181928593.png" alt="image-20230419181928593" style="zoom:50%;" />



## 四、Renderpasses和FrameBuffer

### 4.1 Renderpasses

在Vulkan中，绘制命令被组织成Render Pass。Renderpass是一组subpass的集合，**每个subpass描述的是如何使用color attachments等图像资源。**Renderpass管理着subpass之间的依赖关系和绘制顺序。

<img src=".asserts/image-20230419181946319.png" alt="image-20230419181946319" style="zoom:50%;" />

### 4.2 FrameBuffer

**FrameBuffer是一系列Image对象的集合，它是所有绘制操作的最终目标。**

Swap Chains从窗口系统申请来的Image对象跟FrameBuffer绑定以后，绘制操作才能绘制到这些Image上面。

<img src=".asserts/image-20230419182002845.png" alt="image-20230419182002845" style="zoom:50%;" />

如果把绘制流程比作绘画：

Swap Chains从窗口系统申请来的Image是用来作画的纸张；

FrameBuffer就是一个可以夹住多张纸的画板；

Renderpasses则表示每张纸要画什么，他们之间的绘制顺序是什么样子的，绘制时是不是要参考前一张的绘制内容。

当绘制完成后，我们就可以把Image传给平台的窗口系统进行显示了。

<img src=".asserts/image-20230419182024546.png" alt="image-20230419182024546" style="zoom:50%;" />

**参考文档：**

1. *Vulkan Programming Guide*
2. *Vulkan Cookbook*
3. *Learning Vulkan*
4. *Vulkan® 1.1.148 - A Specification*
5. *Create the Framebuffers*
6. *Create a Swapchain*
7. *API without Secrets: Introduction to Vulkan\* Part 2: Swap Chain*

> *原文链接:* *https://zhuanlan.zhihu.com/p/166423581*