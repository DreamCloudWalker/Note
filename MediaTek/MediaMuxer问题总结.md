## 写入的MP4不能播放

#### MediaMuxer.release()函数必须执行完毕

否则MP4文件缺少moov_box，造成mp4无法播放。

如果ffmpeg写入没有调用av_write_trailer，也会有同样的问题。



## Stop失败

日志报的是状态不对，但断点看状态处于正常状态。

网上很多说是addTrack时机问题，但不是这个问题。

注意看<font color="red">**MPEG4Writer**</font>的日志。

会发现有：E/MPEG4Writer(11768): There are no sync frames for video track 这样的日志。

参考文章：https://mlog.club/article/59907

这里说是：多路复用视频输出必须以同步帧(a / k / a关键帧,a / k / a I帧)开头.如果您从预测帧开始,但是没有什么可预测的,则解码器将不知道该怎么做.

确保将所有[MediaCodec.BufferInfo](https://mlog.club/redirect?url=http%3A%2F%2Fdeveloper.android.com%2Freference%2Fandroid%2Fmedia%2FMediaCodec.BufferInfo.html)值都传递给MediaMuxer,这是标志所在的位置.同步帧将设置BUFFER_FLAG_SYNC_FRAME标志.

(更新：从API 21开始,不推荐使用BUFFER_FLAG_SYNC_FRAME,而建议使用BUFFER_FLAG_KEY_FRAME.两个符号都具有相同的整数值和相同的含义；更改只是在API中采用一致术语的一部分.)



我这边的情况是：

导出的pts是自己计算的，依次调用encode函数，从0开始递增。但后面开始writeSampleData的时候，第一帧是Config帧，用了pts=0。后面的帧就是从pts=3333开始了。而视频最后一针是关键帧，pts变成了0。导致无法正常Stop掉MediaMuxer。

解决方案：把调encode的pts入队列。后面写的帧看是不是Config帧，如果是config帧就给pts=0。其他的依次从队列取。



如果要编码B帧，PTS的计算可能还要另外考虑。