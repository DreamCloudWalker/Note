## 一、使用

1. 创建单通道纹理

```c++
GLuint id_ = 0;
glGenTextures(1, &id_);
glBindTexture(GL_TEXTURE_2D, id_);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glTexImage2D(GL_TEXTURE_2D, 0, GL_LUMINANCE, width_, height_, 0, GL_LUMINANCE, GL_UNSIGNED_BYTE, data);
glBindTexture(GL_TEXTURE_2D, 0);
```

2. 更新纹理数据

```
glBindTexture(GL_TEXTURE_2D, id_);
glTexSubImage2D(GL_TEXTURE_2D, 0, 0, 0, width_, height_, GL_LUMINANCE, GL_UNSIGNED_BYTE, data);
glBindTexture(GL_TEXTURE_2D, 0);
```

### 特别注意：

在更新前后加上如下代码，不然可能因为内存未对齐而出现随机的异常

```c++
glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
// update your texture...

glPixelStorei(GL_UNPACK_ALIGNMENT, 4);
```

`glPixelStorei()`函数用于设置像素存储模式。它可以影响到后续像素操作函数的行为，如`glReadPixels()`、`glTexImage2D()`等。

`glPixelStorei()`函数有两个参数：`pname`和`param`。其中，`pname`指定要设置的像素存储模式，`param`指定对应的值。

以下是一些常用的`pname`参数及其作用：

- `GL_UNPACK_ALIGNMENT`：设置像素数据在内存中的对齐方式。默认值是4，表示每一行像素数据在内存中的地址对齐到4字节边界。可以通过设置`param`为1、2、4或8来改变对齐方式。
- `GL_PACK_ALIGNMENT`：设置像素数据在内存中的对齐方式。默认值是4，表示每一行像素数据在内存中的地址对齐到4字节边界。可以通过设置`param`为1、2、4或8来改变对齐方式。
- `GL_PACK_ROW_LENGTH`：设置读取像素数据时每一行的像素数目。默认值为0，表示每一行的像素数目与纹理的宽度相同。可以通过设置`param`来改变每一行的像素数目。
- `GL_PACK_SKIP_PIXELS`：设置读取像素数据时跳过的像素数目。默认值为0，表示从第一个像素开始读取。可以通过设置`param`来跳过指定数量的像素。
- `GL_PACK_SKIP_ROWS`：设置读取像素数据时跳过的行数。默认值为0，表示从第一行开始读取。可以通过设置`param`来跳过指定数量的行。

`glPixelStorei()`函数通常在准备读取或写入像素数据之前调用，以确保像素数据按照所需的方式存储和访问。例如，如果要读取一个纹理的像素数据，可以使用`glPixelStorei(GL_PACK_ALIGNMENT, 1)`来设置像素数据的对齐方式为1字节，以便按照字节对齐的方式读取像素数据。



## 二、Dump

