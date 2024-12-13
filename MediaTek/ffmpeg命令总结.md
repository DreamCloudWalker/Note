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
* h265互转h264
    * ffmpeg -i inputfile.mp4 -map 0 -c:a copy -c:s copy -c:v libx264 output.mp4
    * ffmpeg -i H264.mp4 -map 0 -c copy -c:v:0 libx265 -y H265.mp4          // 保留所有音频轨、字幕轨等等并且直接复制，仅仅转换视频轨0为H265
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



### **分割视频**： 

假设你知道视频总长，比如说是 6 分钟（360 秒），那么你可以使用以下命令将其分割为两个部分：

```shell
# 第一个文件
ffmpeg -i input.mp4 -ss 0 -t 180 -c copy part1.mp4

# 第二个文件
ffmpeg -i input.mp4 -ss 180 -c copy part2.mp4
```



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

* 查看关键帧信息

```shell
ffprobe -v error -select_streams v:0 -show_entries frame=pict_type -of csv=print_section=0 input.mp4
```

打印结果如下：

```
I,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,P,
```

或者用

```
ffprobe -v quiet -print_format json -show_format -show_streams input.mp4
// 或
ffprobe -v quiet -print_format json -show_format -show_frames input.mp4
```

打印出结果如下：

```shell
NK2M63R7VN:images jian.deng$ ffprobe -v quiet -print_format json -show_format -show_frames shopee_23.mp4 
{
    "frames": [
        {
            "media_type": "video",
            "stream_index": 0,
            "key_frame": 1,
            "pts": 0,
            "pts_time": "0.000000",
            "pkt_dts": 0,
            "pkt_dts_time": "0.000000",
            "best_effort_timestamp": 0,
            "best_effort_timestamp_time": "0.000000",
            "pkt_pos": "48",
            "pkt_size": "8739",
            "width": 1440,
            "height": 1280,
            "pix_fmt": "yuv420p",
            "sample_aspect_ratio": "1:1",
            "pict_type": "I",
            "coded_picture_number": 0,
            "display_picture_number": 0,
            "interlaced_frame": 0,
            "top_field_first": 0,
            "repeat_pict": 0,
            "color_range": "tv",
            "color_space": "bt709",
            "color_primaries": "bt709",
            "color_transfer": "bt709",
            "chroma_location": "left",
            "side_data_list": [
                {
                    "side_data_type": "H.26[45] User Data Unregistered SEI message"
                }
            ]
        },
        {
            "media_type": "video",
            "stream_index": 0,
            "key_frame": 0,
            "pts": 3000,
            "pts_time": "0.033333",
            "pkt_dts": 3000,
            "pkt_dts_time": "0.033333",
            "best_effort_timestamp": 3000,
            "best_effort_timestamp_time": "0.033333",
            "pkt_pos": "16134",
            "pkt_size": "1005",
            "width": 1440,
            "height": 1280,
            "pix_fmt": "yuv420p",
            "sample_aspect_ratio": "1:1",
            "pict_type": "B",
            "coded_picture_number": 3,
            "display_picture_number": 0,
            "interlaced_frame": 0,
            "top_field_first": 0,
            "repeat_pict": 0,
            "color_range": "tv",
            "color_space": "bt709",
            "color_primaries": "bt709",
            "color_transfer": "bt709",
            "chroma_location": "left"
        },
        {
            "media_type": "video",
            "stream_index": 0,
            "key_frame": 0,
            "pts": 6000,
            "pts_time": "0.066667",
            "pkt_dts": 6000,
            "pkt_dts_time": "0.066667",
            "best_effort_timestamp": 6000,
            "best_effort_timestamp_time": "0.066667",
            "pkt_pos": "14235",
            "pkt_size": "1899",
            "width": 1440,
            "height": 1280,
            "pix_fmt": "yuv420p",
            "sample_aspect_ratio": "1:1",
            "pict_type": "B",
            "coded_picture_number": 2,
            "display_picture_number": 0,
            "interlaced_frame": 0,
            "top_field_first": 0,
            "repeat_pict": 0,
            "color_range": "tv",
            "color_space": "bt709",
            "color_primaries": "bt709",
            "color_transfer": "bt709",
            "chroma_location": "left"
        },
        {
            "media_type": "video",
            "stream_index": 0,
            "key_frame": 0,
            "pts": 9000,
            "pts_time": "0.100000",
            "pkt_dts": 9000,
            "pkt_dts_time": "0.100000",
            "best_effort_timestamp": 9000,
            "best_effort_timestamp_time": "0.100000",
            "pkt_pos": "17139",
            "pkt_size": "1376",
            "width": 1440,
            "height": 1280,
            "pix_fmt": "yuv420p",
            "sample_aspect_ratio": "1:1",
            "pict_type": "B",
            "coded_picture_number": 4,
            "display_picture_number": 0,
            "interlaced_frame": 0,
            "top_field_first": 0,
            "repeat_pict": 0,
            "color_range": "tv",
            "color_space": "bt709",
            "color_primaries": "bt709",
            "color_transfer": "bt709",
            "chroma_location": "left"
        },
```

这里找"key_frame": 1,的，可以看出I帧间隔