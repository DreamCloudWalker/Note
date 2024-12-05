## ActvityManagerServcie的重要功能

start(): 1)启动 CPU 监控线程; 2) 注册电池状态和权限管理服务 

startObservingNativeCrashes()： 监听所有的crash事件 

setSystemProcess()： 添加各种管理app状态信息的服务还有进程等等信息



## SystemServer进程注意事项

<img src=".asserts/image-20241121102542242.png" alt="image-20241121102542242" style="zoom:50%;" />



## Application启动流程分析

![image-20241121105050664](.asserts/image-20241121105050664.png)