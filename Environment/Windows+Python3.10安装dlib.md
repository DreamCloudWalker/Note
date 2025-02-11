先在环境终端运行以下两个指令：

1.     pip install cmake

2.     pip install boost
   
   
   其次，找到你的python版本对应的dlib，下载到本地
   
   [GitHub - datamagic2020/Install-dlib](https://github.com/datamagic2020/Install-dlib/tree/main)

左下角蓝色区域点击下载

我的是python 3.10 对应的dlib的whl文件是--win

dlib-19.22.99-cp310-cp310-win_amd64.whl



最后在也是在环境终端

先进入目标文件夹

 cd F:\study（dlib-19.22.99-cp310-cp310-win_amd64.whl）
然后再 

pip install dlib-19.22.99-cp310-cp310-win_amd64.whl
如果用于人脸检测项目还需要import cv2

pip install -i https://pypi.tuna.tsinghua.edu.cn/simple opencv-python



