现在安装brew，一会报这个错，一会儿报那个错，上网查了很多教程，用了很多时间都是不可以，电脑开VPNFQ也不行。

```
Warning: The Ruby Homebrew installer is now deprecated and has been rewritten in
Bash. Please migrate to the following command:
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused
```

无论怎么弄，都下载不到，我直接从GitHub上面克隆，不可以。然后叫我用官网的命令，我用官网的命令下载，又说curl什么的访问不到。后来下载保存了一个brew_install.rb文件，通过ruby brew_install.rb下载，也不行，回过头来又叫我用官网命令下载。在网上看到说要改hosts文件，我也改了，都不可以。



# 解决

不过看到一个大佬放上一条命令，直接解决了这一系列问题，可以下载。很感谢这位大佬，不多说直接上命令：

自动脚本(全部国内地址)（在Mac os终端中复制粘贴回车下面这句话)

```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

这行命令就是这位大佬发布的：[金牛肖马](https://zhuanlan.zhihu.com/p/111014448)