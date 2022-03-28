```shell
adb shell    
cd data/anr
ls -a      //列出所有文件
```

如果 有root权限，可以用adb pull或mv等命令拷贝出来。如果提示权限拒绝，可以用 ***adb bugreport*** 

此命令会导出一个zip压缩包，解压后在本地目录下就可以看到traces文件了。

- adb bugreport 命令也可以指定文件导出目录，如导出到桌面：adb bugreport C:\Users\Nxin\Desktop ；不指定时会在当前命令行对应目录下导出压缩包。



