### Mac安装
* brew install ffmpeg



### ffplay

播放原始视频yuv数据, 以1280*720的xxx.yuv为例
* ffplay -f rawvideo -video_size 1280x720 xxx.yuv

播放48kHz 单声道 16bit的test_decoder.pcm的PCM文件为例
* ffplay -ar 48000 -channels 2 -f s16le -i test_decoder.pcm



### 格式转换

* ffmpeg -i input.mp4 output.avi
* ffmpeg -pix_fmt yuv420p -s 960x540 -i little_prince.jpg little_prince_yuv420p_960x540.yuv
    * -pix_fmt：表示要用什么格式转换，yuv420p是参数，你也可以通过ffmpeg -pix_fmts查看其它支持的类型。
    * -s：表示一帧的尺寸，这个尺寸将是你生成yuv数据的宽和高，需要牢牢记住，因为转换成yuv数据后，数据将不会存储任何无关信息，包括尺寸。
    * -i：很简单，就是需要转换的文件，i表示input。参数不仅可以是jpg格式的，也可以是bmp等任何其它常见类型。
    * little_prince_yuv420p_960x540.yuv：我的输出文件名，在文件名中，记录了yuv的格式，和尺寸，这些信息在显示过程中比较重要。
* h265转h264
    * ffmpeg -i inputfile.mp4 -map 0 -c:a copy -c:s copy -c:v libx264 output.mp4
* 设置B帧
    * -bf 0 主要设置B帧数量
    * keyint=1:min-keyint=1 主要设置 I帧间隔 和最小I帧间隔，其实就是GOP数量

```
ffmpeg -i testIn.mp4 -vcodec libx264 -bf 0 -g 1 -x264-params  "keyint=1:min-keyint=1" -crf 26 -an -y testOut.mp4
```
* 插入B帧
    * bframes ：设定I帧与P帧之间的最大B帧数量，范围0~16
    * b-adapt ： B帧自适应方法，默认为1
        * 0: Disabled
        * 1: Fast
        * 2: Optimal (slow with high --bframes)
    * -sc_threshold ：转场时插入B帧
    * -g 设置GOP，每50帧插入一个关键帧



### 给mp3添加封面

ffmpeg -i 别云间-明-夏完淳.mp3 -i 别云间.jpg -map 0:0 -map 1:0 -c copy -id3v2_version 3 -metadata:s:v title="Album cover" -metadata:s:v comment="Cover (Front)" 别云间-明-夏完淳F.mp3



### ffprobe

* 统计视频I、B、P帧: 

```shell
ffprobe -v quiet -show_frames transcoded123.mp4 | grep "pict_type=B" | wc -l
```

控制台输出0，表示这个视频没有B帧，如果要查看I帧和P帧，修改pict_type=I或者P即可
如果需要看关键帧，grep “key_frame=1”。

* 查看文件真实格式

```shell
ffprobe -show_frames test.jpg
```

* 查看包信息

```shell
ffprobe -show_packets input.mp4
```

Packets信息与帧信息比较类似，也是包括pts信息、大小等。