byte数据判断是否I帧（关键帧）：

```c++
h264 :
uint8_t nal_type = (nal_start[0] & 0x1F);
hevc（h265）:
uint8_t nal_type = (nal_start[0] & 0x7E) >> 1;
```

比如byte数据是0x65, 0x65 & 0x1F == 5，代表I帧；



avcC第一帧的数据前8个字节表示数据长度，比如demux打印出来的数据前面是：

00 00 45 7C 65 B8

0x457C = 17788，表示数据总大小。再加上4个字节，为17792。





打印char *或uint8_t *数据，或在java端打印byte[]或ByteBuffer数据。

Uint8_t *data;   

printf("%02X", data[0]);

ByteBuffer data;

Log.d(TAG, String.format("data = %02X", data.get(0)););