项目的安卓视频硬解，是通过C++层通过jni回调MediaCodec的Java接口去实现的。

C++层从ffmpeg的demuxer拿到数据，开始要将config数据配置给MediaCodec，即flags=2，pts = 0，调用mDecoder.queueInputBuffer传入。

但发现在特定手机<font color="red">（Redmi 9C，这个手机出过无数硬解兼容问题）</font>上某个特定视频，C++传的是flag=2, pts=0，但java接收到的变成了pts=8589934592,flags=10000。查了好久才发现，是因为C++层pts直接传了0，在java层被当做了一个int。导致取参数错乱。Jni注册和传参如下：

C++代码

```c++
// jni动态注册
{s_method_decode.c_str(),           "(Ljava/nio/ByteBuffer;JIJ)I",           false},

// decode
jmethodID decode_method_id = FIND_METHOD(mediacodec::s_class_name, mediacodec::s_method_decode);
if (!decode_method_id) {
  MAKE_ERROR(error, kErrorCodeNullptr, "MediaCodecDecoder::init decode_method_id is nullptr!");
  return;
}

if (!config.extra_data || !config.extra_data.get() || config.extra_data->size() <= 0) {
  MAKE_ERROR(error, kErrorCodeDecodeConfigure, "MediaCodecDecoder::init config.extra_data is wrong!");
  return;
}
std::shared_ptr<Data> config_data = BitStreamUtils::H264ExtraData2Annexb(config.extra_data);
if (!config_data || !config_data.get()) {
  MAKE_ERROR(error, kErrorCodeDecodeConfigure, "MediaCodecDecoder::init h264ExtraData2Annexb failed!");
  return;
}
int current_count = 0;
ScopedLocalRef<jobject> config_buffer(env, data_to_byte_buffer(env, config_data->data(), config_data->size()));
int ret = env->CallIntMethod((*class_obj_)(), decode_method_id, config_buffer(), 0, kBufferFlagConfig, kConfigTimeOutUs);	// 注意这里，这里直接传了0，其中kBufferFlagConfig是int型的0，kConfigTimeOutUs是long型的10000
```

Java代码

```java

  // flags: BUFFER_FLAG_CODEC_CONFIG = 2; BUFFER_FLAG_END_OF_STREAM = 4; BUFFER_FLAG_KEY_FRAME = 1;
public @SSPEditorMediaCodecErrorType int decode(ByteBuffer data, long presentationTimeUs, int flags, long timeoutUs) {
  ...
}
```

因为C++层pts直接传了0，在java层被当做了一个int。导致参数错乱后，Java接收到的pts=8589934592,flags=10000。导致config失败。但MediaCodec的解码还在继续进行。但取出来的帧乱序了，导致缓存的阻塞队列里第一帧的pts=66667，第二帧是33333。

而从缓存阻塞队列的取帧逻辑，发现第一帧pts>第二帧，后面进来的帧也出现乱序。逻辑里一直会取一帧，并且不pop，导致解码出来只有一帧画面。



最后改成了如下代码就恢复正常了

```c++
int ret = env->CallIntMethod((*class_obj_)(), decode_method_id, config_buffer(), (int64_t)0, kBufferFlagConfig, kConfigTimeOutUs);	// 注意这里，这里直接传了0
```













