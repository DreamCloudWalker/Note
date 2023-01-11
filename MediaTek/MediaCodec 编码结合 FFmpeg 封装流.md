在 Android 平台上合成视频一般使用 MediaCodec 进行硬编码，使用 MediaMuxer 进行封装，但是因为 MediaMuxer 在某些机型上合成的视频在其他手机上播放会出现问题，而且只支持一个音频轨道，因此可以选**用 FFmpeg 来封装编码后的音视频流**。

具体流程如下：

## 1. 创建FFmpeg AVFormatContext

```
AVFormatContext*ofmt_ctx = nullptr;
intret = avformat_alloc_output_context2(&ofmt_ctx, nullptr, "mp4", filePath);
AVOutputFormat*ofmt = ofmt_ctx->oformat;
ret = avio_open(&ofmt_ctx->pb, filePath, AVIO_FLAG_WRITE);
```



## 2. 添加音视频流

这里以添加 h264 视频流为例:

```c++
AVStream*stream = avformat_new_stream(ofmt_ctx, nullptr);
intvideo_stream = stream->index;
AVCodecParameters*codecpar = stream->codecpar;
codecpar->codec_type = AVMEDIA_TYPE_VIDEO;
codecpar->codec_id = AV_CODEC_ID_H264;
codecpar->width = width;
codecpar->height = height;
```



## 3. 设置视频流sps和pps

sps 和 pps 能在 MediaCodec 产生第一帧画面之前获取到，以 java MediaCodec异步编码方式为例

java 接口获取 sps 和 pps 数据

```java
@Override
public void onOutputBufferAvailable(@NonNullMediaCodeccodec, intindex, @NonNullMediaCodec.BufferInfoinfo) {
ByteBuffer buffer=encoder.getOutputBuffer(index);
if ((info.flags&MediaCodec.BUFFER_FLAG_CODEC_CONFIG) !=0) {
// 传递 buffer和info.size到native
    }
// ...
}
```

native 获取 sps 和 pps 数据地址

```c++
uint8_t*data = static_cast<uint8_t*>(env->GetDirectBufferAddress(buffer));
// 复制给视频流extradata
AVCodecParameters*codecpar = ofmt_ctx->streams[video_stream]->codecpar;
codecpar->extradata = (uint8_t*) av_mallocz(size+AV_INPUT_BUFFER_PADDING_SIZE);
memcpy(codecpar->extradata, data, size);
codecpar->extradata_size = size;
```



## 4. 写入视频文件头信息，放在文件开头位置

```c++
AVDictionary*dict=nullptr;
av_dict_set(&dict, "movflags", "faststart", 0);
intret=avformat_write_header(ofmt_ctx, &dict);
```



## 5. 写入视频流和音频流已编码数据

同样以写入视频流数据为例，⚠️注意视频流和音频流在不同线程写入时需要同步

onOutputBufferAvailable 回调中

```
booleanisKeyFrame= (info.flags&MediaCodec.BUFFER_FLAG_KEY_FRAME) !=0;
// 传递buffer, info.size, isKeyFrame, info.presentationTimeUs到native
```

获取视频编码数据地址

```
uint8_t *data = static_cast<uint8_t *>(env->GetDirectBufferAddress(buffer));

AVPacket *packet = av_packet_alloc();
av_init_packet(packet);
packet->stream_index = video_stream;
packet->data = data;
packet->size = size;
packet->pts = av_rescale_q(pts, { 1, 1000000 }, ofmt_ctx->streams[video_stream]->time_base);
if (isKeyFrame) packet->flags |= AV_PKT_FLAG_KEY;

int ret = av_interleaved_write_frame(ofmt_ctx, packet);
av_packet_unref(packet);
av_packet_free(&packet);
```

## 6. 结束并关闭文件

```c++
av_write_trailer(ofmt_ctx);
avio_closep(&ofmt_ctx->pb);
avformat_free_context(ofmt_ctx);
```

至此，整个流程就结束了。

作者：VE 视频引擎

来源：https://segmentfault.com/a/1190000039402914

