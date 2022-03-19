升级 Big Sur 后发现JD-GUI 打开报错：

> ERROR launching 'JD-GUI'
>
> No suitable Java version found on your system!
>  This program requires Java 1.8+
>  Make sure you install the required Java version.

选择直接删除 JD-GUI ，然后在官网重新下载了 最新版本 [http://java-decompiler.github.io/](https://links.jianshu.com/go?to=http%3A%2F%2Fjava-decompiler.github.io%2F)
 [**jd-gui-osx-1.6.6.tar**](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fjava-decompiler%2Fjd-gui%2Freleases%2Fdownload%2Fv1.6.6%2Fjd-gui-osx-1.6.6.tar)
 发现仍然报同样的错误。

参见 JD-GUI 的issue ：[Update universalJavaApplicationStub to be able to launch on macOS Big Sur #336](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fjava-decompiler%2Fjd-gui%2Fpull%2F336)
 我们需要替换一个文件；universalJavaApplicationStub.sh (version 3.0.6)

将此文件内容替换为 [https://github.com/tofi86/universalJavaApplicationStub/blob/v3.0.6/src/universalJavaApplicationStub](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ftofi86%2FuniversalJavaApplicationStub%2Fblob%2Fv3.0.6%2Fsrc%2FuniversalJavaApplicationStub) 中的内容；

保证系统上正确安装了 Java ；
 保存，运行 JD-GUI.app OK。



dex2jar地址：https://github.com/pxb1988/dex2jar