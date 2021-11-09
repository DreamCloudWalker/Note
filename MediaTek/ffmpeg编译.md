### 1. 参考网上的老编译脚本，会报出

ffbuild/config.mak: No such file or directory

等问题。最后发现是脚本需要多一句：--disable-x86asm

且在这之前，需要把configure文件中其中4行改为(这段主要作用是编译之后的库的名称，让同时存在带版本号的和不带版本号的库)：

```sh
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'

LIB_INSTALL_EXTRA_CMD='$$(RANLIB) “$(LIBDIR)/$(LIBNAME)”'

SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'

SLIB_INSTALL_LINKS='$(SLIBNAME)'
```



**总体脚本如下：**

```sh
# 设置编译后文件的输出目录
CPU=armv7-a
OUTPUTDIR=$(pwd)/android/$CPU
ADDI_CFLAGS="-marm"
# 当前设置为最低支持android-21版本，arm架构
API=21
PLATFORM=arm-linux-androideabi
NDK="/Users/jian.deng/SDK/android-ndk-r16b"
# 设置编译工具链，4.9为版本号
TOOLCHAIN=$NDK/toolchains/$PLATFORM-4.9/prebuilt/darwin-x86_64/bin/arm-linux-androideabi-
PLATFORM=$NDK/platforms/android-$API/arch-arm/
ISYSROOT=$NDK/sysroot
ASM=$ISYSROOT/usr/include/$PLATFORM
function build_ffmpeg
{
pwd
./configure \
--prefix=$OUTPUTDIR \
--enable-cross-compile \
--enable-shared \
--disable-static \
--disable-doc \
--disable-x86asm
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffserver \
--disable-avdevice \
--disable-symver \
--cross-prefix=$TOOLCHAIN \
--target-os=android \
--arch=arm \
--extra-cflags="-I$ASM -isysroot $ISYSROOT -0s -fpic $ADDI_CFLAGS" \
--extra-ldflags="$ADDI_CFLAGS" \
#--host=aarch64-linux-gnu \ 
$ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install
}
build_ffmpeg
```

但mac下编译完成后生成的是.dylib，而不是.so

### 2. 解决mac下编译完成后生成的是.dylib，而不是.so



### 3. 编译成一个so