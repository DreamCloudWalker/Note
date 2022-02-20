## 以JD-GUI的使用为例

参考https://sqiang.net/post/2524245389.html

JD-GUI 是一款大家耳熟能详的 Java 反编译工具，可以方便的将编译好的 `.class` 文件反编译为 `.java` 源码文件，用于开发调试、源码学习等。

官网地址：[http://java-decompiler.github.io](http://java-decompiler.github.io/)

Git 地址：https://github.com/java-decompiler/jd-gui



### 解决在 macOS 下闪退问题

需要注意的是，**运行 JD-GUI 所需 Java 版本最高为 JDK 10.0.2（可在官网文档查看）**，否则会出现闪退、无法使用等问题，所以需要修改设置进行指定，下面以 macOS 为例进行说明。

首先，使用如下命令，查看 jdk 的安装路径：

```
# 因为我只安装了 Java1.8 和 Java11 两个版本，所以这里需要找 1.8 的路径
➜ /usr/libexec/java_home -v 1.8
/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home
```

其次，使用文本编辑器打开文件 `/Applications/JD-GUI.app/Contents/MacOS/universalJavaApplicationStub.sh`，修改如下部分：

```
# first check system variable "$JAVA_HOME"
# 这里的配置默认找的是系统环境变量中配置的 jdk 版本
# 而我配置的是 Java11，所以直接运行的时候出现闪退而无法使用
# 这里需要直接写死 1.8 的路径
if [ -n "$JAVA_HOME" ] ; then
	JAVACMD="/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/bin/java"
```

保存后，重新打开 JD-GUI.app 即可。

