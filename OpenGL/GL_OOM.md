报错如下：

<gsl_ldd_control:553>: ioctl fd 95 code 0xc0300945 (IOCTL_KGSL_GPUOBJ_ALLOC) failed: errno 12 Out of memory 

<gsl_memory_alloc_pure:2604>: GSL MEM ERROR: kgsl_sharedmem_alloc ioctl failed.



有些条件也会出现glError: 1285

public static final int GL_OUT_OF_MEMORY = 0x0505;



而Profiler或adb dumpsys meminfo看都没有内存和显存泄露。后逐步缩小范围，比如先去掉Pipeline的几个节点，发现只有上屏节点没问题。后又发现上屏节点如果增加FBO也会出问题。才发现FBO对象判断条件出错，导致每帧都在glGenFramebuffers。导致显存碎片最后出现OOM。
