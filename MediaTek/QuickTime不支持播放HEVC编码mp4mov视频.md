参考： https://blog.csdn.net/solan8/article/details/116300899

https://juejin.cn/post/6854573210579501070

编码出来的H265视频，会发现有些在mac上图标可以预览，用quicktime player可以播放，但有些不行。



用ffprobe或者mediainfo分析相关视频文件，会发现是hev1。

hev1、hvc1是两种codec tag。Quicktime Player和iOS不再支持hev1 tag的mp4/mov。

二者大致有如下不同：

'hvc1' stores all parameter sets inside the MP4 container below the sample description boxes.
'hev1' stores all parameter sets in band (inside the HEVC stream).
参考下例，用FFmpeg生成QuickTime兼容HEVC码流

ffmpeg -i src/8k_265.mov -c:v copy -vtag hvc1 8k_265.mp4



https://blog.csdn.net/iBobbyTS/article/details/109402983

https://stackoverflow.com/questions/63468587/what-hevc-codec-tag-to-use-with-fmp4-hvc1-or-hev1