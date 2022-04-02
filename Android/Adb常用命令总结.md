### 1. adb常用（杂） 

  adb shell

  adb top 详见top命令总结笔记

  adb devices

  adb reboot

  adb remount 类似adb root

  adb shell am force-stop com.package.packagename   // 杀进程

  adb shell screencap -p /sdcard/screencap.png && adb pull /sdcard/screencap.png C:\screenshot.png	// 截屏

  pm -lf|grep "aruler"	// 查找（比如查找包在哪个目录）

* adb shell **getprop** 
  * adb shell **getprop** ro.hardware 	// dump CPU信息
  * 查看设备相关信息，都可用，后面参数可参考Build.java。比如查看手机model信息，就用 adb shell getprop ro.product.model

* root后
  * adb disable-verity
  * adb shell getprop ro.product.cpu.abi 	// 查看手机arm版本

*  find
  * find / -name xxx.xxx	// 按文件名在**根目录**查找
  * find frameworks/ -name xxx.xxx    // 在frameworks目录下查找
  * find frameworks/ -name ‘PowerManager*’   // 使用通配符*（0个或任意多个），在frameworks目录下查找文件名开头是字符串‘PowerManager’的文件
  * find . -name ‘PowerManager*’   // 表示在当前目录下（包含子目录）查找文件名开头是字符串‘PowerManager’的文件。
  * find frameworks/ -amin -10    // 按文件特征找，表示在frameworks目录下查找最后10分钟访问的文件。

* grep
  * -i：不区分大小写；
  * -n：显示匹配行及行号；
  * -r：包含子目录；
  * -c：只输出匹配行的计数；
  * -w：匹配整个单词；



  查看root手机里的pref等文件

  adb root后，adb shell，cd到data/data/com.your.packagename这个目录，ls可查看文件

  如果要拷贝出这里面的文件，可先把他拷贝到手机SD卡，比如再data目录下，cp 1.jpg /mnt/sdcard/

  

### 2. 进程线程信息

adb shell后 

\>> top -H	// 查看活跃进程，默认按cpu占用率排序，如图：

![image-20210923174245910](.asserts/image-20210923174245910.png)

第一个是进程号，可进一步

\>> top -H -p <pid>

![image-20210923174618280](.asserts/image-20210923174618280.png)

也可以用代码打印出来：

```java
		private void printThread() {
        Map<Thread, StackTraceElement[]> stacks = Thread.getAllStackTraces();
        Set<Thread> set = stacks.keySet();
        for (Thread key : set) {
            StackTraceElement[] stackTraceElements = stacks.get(key);
            Log.d(TAG, "---- print thread: " + key.getName() + " start ----");
            for (StackTraceElement st : stackTraceElements) {
                Log.d(TAG, "StackTraceElement: " + st.toString());
            }
            Log.d(TAG, "---- print thread: " + key.getName() + " end ----");
        }
    }
```



// 定位线程泄漏

先用**ps -fe |grep programname** 查看进程id

adb shell cat /proc/<pid>/status 	// 如图的Threads:后面标注了线程数量

![image-20210923175903909](.asserts/image-20210923175903909.png)



### 3. 查看activity信息

adb shell dumpsys activity 	// adb查看当前task的activity

adb shell dumpsys activity | grep 应用的package

adb shell dumpsys activity | grep mFocusedActivity

**adb -d shell dumpsys activity activities | grep mResumedActivity** // 查看当前显示的activity

**adb shell dumpsys activity activities | sed -En -e '/Running activities/,/Run #0/p'**	// 查看activity堆栈

adb shell dumpsys activity top  // 可以看到Added Fragment的信息



### 4. adb logcat

adb logcat | grep -E "keyword1|keyword2$"

比如 adb logcat | grep -E "arworks|arcore|OAR-E"

输出到电脑上

adb logcat > e:\log\log.txt



### 5. 查看到内存详情，包括虚拟机内存、Native内存，图层信息等

adb shell dumpsys meminfo packageName

adb dumpsys SurfaceFlinger packageName	// 可查看到图层信息，比如：

![image-20210923173624463](.asserts/image-20210923173624463.png)

这上面可以看到当前有多少个图层处于显示状态。如果有卡顿现象，也可以优先查看是否SurfaceView等是否设置了透明属性。导致很多额外图层显示。HWC为图层



dumpsys gfxinfo packagename	// 查看硬件加速是否开启，如果开启，能看到TextureCache，LayerCache等log信息



### Top命令总结

adb shell top -d 1 | grep "com.dengjian.xxx"	// 查看线程的CPU占用率、线程状态、占用内存大小等信息。

-d 表示显示多少条，后面的数字是显示条数。

grep 后是线程名，用作过滤。

但这样看的不够清晰，可以运行这个后，找到对于的进程号，然后用 <font color="red">top -d 1 -p xxxxx</font> , 运行结果如下：

![image-20220401163356515](.asserts/image-20220401163356515.png)

旧版本top输出的数据含义：

```shell
PID   进程id
USER  用户名
PR    优先级
CPU%  当前瞬时CPU占用率
S     进程状态:D=不可中断的睡眠状态, R=运行, S=睡眠, T=跟踪/停止, Z=僵尸进程
#THR  程序当前所用的线程数
VSS   Virtual Set Size  虚拟耗用内存（包含共享库占用的内存）
RSS   Resident Set Size 实际使用物理内存（包含共享库占用的内存）
PCY   调度策略优先级，SP_BACKGROUND/SP_FOREGROUND
UID   进程所有者的用户id
Name  进程的名称
```

新版本为：

![image-20220401164050800](.asserts/image-20220401164050800.png)

