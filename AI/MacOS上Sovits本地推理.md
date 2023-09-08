参考https://xiaoao.cyou/m1-mac%E4%BD%BF%E7%94%A8so-vits-svc-4-0/

这个文章上是用conda环境下的。现在公司不让用conda，下面是我自己的流程。



## **环境搭建**

git clone项目文件https://github.com/svc-develop-team/so-vits-svc

cd到这个目录，pip3 install -r requirements.txt

我原来MacOS默认的python3的版本是3.11.4，这个安装依赖一直报错

ERROR: Could not build wheels for fairseq, which is required to install pyproject.toml-based projects

网上的方案是修改pyworld的版本，比如把requirements.txt这里的pyworld指定版本删掉再安装。

但我是改了python3版本来安装的。



mac安装指定版本python参考：https://zhuanlan.zhihu.com/p/373853381

```text
brew install python@3.10
```

我安装了python3.10版本后（3.10.3），用pip3.10 install -r requirements.txt命令重新安装。这次报错是：

<font color="red">ERROR: Could not build wheels for pyworld, which is required to install pyproject.toml-based projects</font>

然后继续安装pip3.10 install fairseq==0.12.2后，再运行pip3.10 install -r requirements.txt 

终于全部成功了。



下载“必要模型文件”[checkpoint_best_legacy_500.pt](https://ibm.ent.box.com/shared/static/z1wgl1stco8ffooyatzdwsqn2psd9lrr)放入项目的hubert文件夹（推理需要用到，链接失效的话请自行google）。



## 运行

目录下python3.10 webUI.py。就可以推理了。

![image-20230908115104648](.asserts/image-20230908115104648.png)

