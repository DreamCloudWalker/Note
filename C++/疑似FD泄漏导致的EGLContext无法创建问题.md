1. 问题日志：W < gsl_ldd_control:541>: ioctl fd 124 code 0xc0080913 (IOCTL_KGSL_DRAWCTXT_CREATE) failed: errno 28 No space left on device
2. 出问题后eglCreateContext会失败，eglGetError为EGL_BAD_ALLOC                     0x3003。后面会无法创建gl环境，一直黑屏。直到杀进程重启。
3. 用脚本反复进出页面能复现，python脚本如下：

```python
import os
import time

# device = "988b5a374d41314738"
# device = "1fda1d22"  # 小米8SE
# device = "8AHY0L1PW" # Pixel3XL
device = "812ee6d4" # Oppo A7
# device = "Q9T4C19131002075"  # 华为Y9

demo_x_y = {
    "1fda1d22": {  # 小米8SE
        "entrance": [540, 2060]
    },
    "8AHY0L1PW": {
        "entrance": [710, 2715]
    },
    "812ee6d4": {
        "entrance": [335, 734]
    },
    "Q9T4C19131002075": {  # 华为Y9（385分）
        "entrance": [540, 2160]
    }
}

x_y = {
    "988b5a374d41314738": {
        "entrance": [1321, 225],
        "entrance_confirm": [720, 2173],
        "exit_confirm": [660, 105]
    },
    "1fda1d22": {  # 小米8SE
        "entrance": [972 + 70 / 2, 115 + 69 / 2],
        "entrance_confirm": [434 + 212 / 2, 1672 + 55 / 2],
        "exit_confirm": [660, 105]
    },
    "1fda1d23": {  # 小米8SE
        "entrance": [972 + 70 / 2, 115 + 69 / 2],
        "entrance_confirm": [434 + 212 / 2, 1672 + 55 / 2],
        "exit_confirm": [660, 105]
    }
}

for i in range(1, 500):
    print("i = %s" % i)
    cmd = "adb -s %s shell input tap %s %s" % (device, demo_x_y[device]["entrance"][0], demo_x_y[device]["entrance"][1])
    print(cmd)
    os.system(cmd)
    print("进入拍摄页...")
    time.sleep(2)
    cmd_back = "adb -s %s shell input keyevent 4" % device
    os.system(cmd_back)
    print("退出拍摄页...\n")

# for i in range(1, 200):
#     print("i = %s" % i)
#     cmd = "adb -s %s shell input tap %s %s" % (device, x_y[device]["entrance"][0], x_y[device]["entrance"][1])
#     os.system(cmd)
#     time.sleep(0.5)
#     cmd_confirm = "adb -s %s shell input tap %s %s" % (device, x_y[device]["entrance_confirm"][0], x_y[device]["entrance_confirm"][1])
#     os.system(cmd_confirm)
#     print("进入拍摄页...")
#     time.sleep(5)
#     cmd_back = "adb -s %s shell input keyevent 4" % device
#     os.system(cmd_back)
#     # cmd_back_confirm = "adb -s %s shell input tap 660 105" % device
#     # time.sleep(1)
#     # os.system(cmd_back_confirm)
#     print("退出拍摄页...\n")

```

4. 最后确定是以下代码导致，没有abandonAudioFocus

```java
public int requestAudioFocus(AudioManager.OnAudioFocusChangeListener listener) {
    // AudioFocus
    int reqResult = 0;
    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
      mAudioFocusRequest = new AudioFocusRequest.Builder(AudioManager.AUDIOFOCUS_GAIN)
      .setAudioAttributes(new AudioAttributes.Builder()
                          .setUsage(AudioAttributes.USAGE_MEDIA)
                          .setContentType(AudioAttributes.CONTENT_TYPE_MOVIE)
                          .build())
      .setAcceptsDelayedFocusGain(true)
      .setOnAudioFocusChangeListener(listener)
      .build();
      reqResult = mAudioManager.requestAudioFocus(mAudioFocusRequest);
    } else {
      reqResult = mAudioManager.requestAudioFocus(listener, AudioManager.STREAM_MUSIC, AudioManager.AUDIOFOCUS_GAIN);
    }
    return reqResult;
  }
```

需要加入：

```java
public int abandonAudioFocus(AudioManager.OnAudioFocusChangeListener listener) {
    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O && null != mAudioFocusRequest) {
      return mAudioManager.abandonAudioFocusRequest(mAudioFocusRequest);
    } else {
      return mAudioManager.abandonAudioFocus(listener);
    }
  }
```

