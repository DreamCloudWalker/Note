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



### 2. 编译.a并链接成一个so

```sh
#!/bin/bash

# NDK path
#change your ndk path here
NDK=~/Library/Android/sdk/ndk/21.0.6113669
# ARCH build switch
BUILD_ARM_V7=1
BUILD_ARM_64=1
# BUILD_X86_32=1
# BUILD_X86_64=1

# Minimum platform
MIN_PLATFORM_32=19
MIN_PLATFORM_64=21

GCC_VERSION=4.9

ROOT_DIR=`pwd`
BUILD_ROOT_DIR="$ROOT_DIR/build"

# x264 source code (https://code.videolan.org/videolan/x264/-/archive/72db4377/x264-72db4377.zip)
X264_DIR="$ROOT_DIR/libx264"
# ffmpeg source code (https://github.com/FFmpeg/FFmpeg/archive/n4.1.4.zip)
FFMPEG_DIR="$ROOT_DIR/FFmpeg-n4.1.4"

OPENH264="$ROOT_DIR/openh264"

OS=`uname`
HOST_ARCH=`uname -m`
if [ "$OS" == "Linux" ]; then
    HOST_SYSTEM=linux-$HOST_ARCH
elif [ "$OS" == "Darwin" ]; then
    HOST_SYSTEM=darwin-$HOST_ARCH
fi
CORES=$(sysctl -n hw.ncpu)

function cleanup
{
    if [ -d "$BUILD_PREFIX" ]; then
        rm -rf "$BUILD_PREFIX"/lib "$BUILD_PREFIX"/include
    fi
}

function prepare {
    TOOLCHAIN_DIR="$BUILD_ROOT_DIR/$ABI/toolchain"
    if [ ! -d "$TOOLCHAIN_DIR" ]; then
        echo "making toolchain.."
        "$NDK/build/tools/make-standalone-toolchain.sh" --verbose \
            --platform="android-$MIN_PLATFORM" \
            --toolchain="$TOOLCHAIN" \
            --install-dir="$TOOLCHAIN_DIR" || exit 1
    fi
    export PATH="$TOOLCHAIN_DIR/bin":$PATH

    SYSROOT="$TOOLCHAIN_DIR/sysroot"


    FIX_BUILD_EXTRA_CFLAGS="-D__ANDROID_API__=$MIN_PLATFORM -U_FILE_OFFSET_BITS"
}

function build_x264 {
    cd $X264_DIR
    echo "configuring x264: $ABI"
    ./configure --cross-prefix="${CROSS_PREFIX}-" \
        --sysroot="$SYSROOT" \
        --prefix="$BUILD_PREFIX" \
        --enable-pic \
        --enable-static \
        --host="$X264_HOST" \
        --extra-cflags="-O3 -fpic $FIX_BUILD_EXTRA_CFLAGS $EXTRA_ARCH_CFLAGS" \
        --extra-ldflags="$EXTRA_ARCH_LDFLAGS" \
        --disable-cli || exit 1

    echo "making x264: $ABI"
    make clean || exit 1
    make -j$CORES || exit 1
    make install || exit 1
    make distclean || exit 1
    echo "making x264 success: $ABI"
    cd $ROOT_DIR
}

function build_openh264 {
    cd $OPENH264
    case $ABI in
        armeabi-v7a )
            ARCH=arm
            ;;
        arm64-v8a )
            ARCH=arm64
            ;;
        x86 )
            ARCH=x86
            ;;
        x86_64 )
            ARCH=x86_64
            ;;
    esac
    # if your ndk >= r18,  NDK_TOOLCHAIN_VERSION=clang should be added
    TARGET_OS=android
    ANDROID_TARGET=android-$MIN_PLATFORM
    echo "build libopenh264 ${ABI} ${ANDROID_TARGET}"
    echo "build libopenh264 ${ABI} output : ${BUILD_PREFIX}"

     make \
      OS=${TARGET_OS} \
      NDKROOT=$NDK \
      TARGET=$ANDROID_TARGET \
      ARCH=$ARCH \
      clean
  make \
      OS=${TARGET_OS} \
      NDKROOT=$NDK \
      TARGET=$ANDROID_TARGET \
      NDKLEVEL=$MIN_PLATFORM \
      ARCH=$ARCH \
      PREFIX=$BUILD_PREFIX \
      -j4 install

    echo "making openh264 success: $ABI"
    cd $ROOT_DIR
}




function copy_ffmpeg_header_file {
    include_dir="$BUILD_ROOT_DIR/$ABI/include"

    copy_hearders=()
    copy_hearders=("config.h")

    echo "copy ffmpeg headers to $include_dir"
    for path in $(find . -name "*.h" -type f); do
        dir=$(dirname $path)
        if [ $dir != "." ]; then
            copy_hearders+=("${path#"./"}")
        fi
    done

    for h in "${copy_hearders[@]}"
    do
        mkdir -p "$(dirname "$include_dir/$h")" || exit 1
        cp $h "$include_dir/$h" || exit 1
    done
    echo "copy ffmpeg headers success"
}

function build_ffmpeg {
    cd $FFMPEG_DIR
    echo "configuring ffmepg: $ABI"

    CONFIG_OPTIONS="--target-os=linux"
    CONFIG_OPTIONS="$CONFIG_OPTIONS $FFMPEG_ARCH_OPTIONS"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-cross-compile"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --cross-prefix=${CROSS_PREFIX}-"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --prefix=$BUILD_PREFIX"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --sysroot=$SYSROOT"

    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-swscale-alpha"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-podpages"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-avdevice"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-postproc"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-filters"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-w32threads"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-shared"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-iconv"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-libxcb"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-bzlib"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-lzma"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-everything"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-ffplay"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-ffprobe"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-debug"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-doc"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-pthreads"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-decoder=mpeg4,h264,aac,hevc"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-encoder=aac,libx264,png"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-parser=h264,aac"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-muxer=mp4"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-demuxer=flv"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-protocol=file"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-gpl"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-libx264"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=scale"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=crop"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=overlay"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=movie"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=alphamerge"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=colorchannelmixer"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=amix"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=aresample"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=aformat"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-bsf=h264_mp4toannexb"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-bsf=hevc_mp4toannexb"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-bsf=aac_adtstoasc"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-demuxer=concat"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=format"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-ffmpeg"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-pic"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-version3"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-zlib"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-avcodec"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-avfilter"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-swresample"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-swscale"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-demuxer=aac"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-demuxer=mov"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-decoder=mjpeg,png"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-demuxer=image2"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-asm"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-runtime-cpudetect"
    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-small"

#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-symver"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-swscale-alpha"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-programs"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-ffmpeg"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-network"
#
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-encoders"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-encoder=libx264,mpeg4,aac,mjpeg,png"
#
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-decoders"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-decoder=h264,mpeg4,hevc,aac,mp3,mjpeg,png"
#
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-muxers"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-muxer=mp4,image2"
#
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-demuxers"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-demuxer=mov,aac,mp3,image2"
#
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-parsers"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-parser=h264,mpeg4video,hevc,aac,png"
#
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-protocols"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-protocol=file,pipe"
#
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-bsfs"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-devices"
#
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-static"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-small"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-pic"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-gpl"   # x264
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-libx264"
#
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-demuxer=flv"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=scale"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=crop"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=overlay"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=movie"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=alphamerge"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=colorchannelmixer"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-filter=format"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-ffmpeg"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-zlib"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-avcodec"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-avfilter"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-swresample"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-swscale"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-asm"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-runtime-cpudetect"
#    CONFIG_OPTIONS="$CONFIG_OPTIONS --enable-version3"


##---------------------------



    # fix ffmpeg B0 confliction with ndk https://github.com/android-ndk/ndk/issues/630#issuecomment-360967820
    CONFIG_OPTIONS="$CONFIG_OPTIONS --disable-linux-perf"

    ./configure $CONFIG_OPTIONS \
        --extra-cflags="-O3 -fPIC $FIX_BUILD_EXTRA_CFLAGS -I$BUILD_PREFIX/include $EXTRA_ARCH_CFLAGS" \
        --extra-ldflags="-L$BUILD_PREFIX/lib $EXTRA_ARCH_LDFLAGS" || exit 1



#        ./configure --prefix=$BUILD_PREFIX --cross-prefix=${CROSS_PREFIX}- \
#        --target-os=linux $FFMPEG_ARCH_OPTIONS --sysroot=$SYSROOT \
#        --extra-cflags="-O3 -fpic $FIX_BUILD_EXTRA_CFLAGS -I$BUILD_PREFIX/include $EXTRA_ARCH_CFLAGS" \
#        --extra-ldflags="-L$BUILD_PREFIX/lib $EXTRA_ARCH_LDFLAGS" \
#        --disable-swscale-alpha \
#        --disable-podpages --disable-avdevice --disable-postproc --disable-filters --disable-w32threads \
#        --disable-shared --disable-iconv --disable-libxcb --disable-bzlib --disable-lzma \
#        --disable-everything --disable-ffplay --disable-ffprobe --disable-debug --disable-doc \
#        --enable-thumb \
#        --enable-pthreads --enable-decoder=mpeg4 --enable-decoder=h264 --enable-decoder=aac \
#        --enable-encoder=aac --enable-encoder=libx264 --enable-parser=h264 --enable-parser=aac \
#        --enable-muxer=mp4 --enable-demuxer=flv --enable-protocol=file --enable-gpl --enable-libx264 --enable-filter=scale \
#        --enable-filter=crop --enable-filter=overlay --enable-filter=movie --enable-filter=alphamerge --enable-filter=colorchannelmixer \
#        --enable-filter=format --enable-ffmpeg --enable-pic --enable-version3 --enable-zlib --enable-avcodec --enable-avfilter \
#        --enable-swresample --enable-swscale --enable-demuxer=aac --enable-demuxer=mov --enable-decoder=mjpeg,png --enable-demuxer=image2 \
#        --enable-asm --enable-runtime-cpudetect --enable-small || exit 1





    echo "making ffmpeg: $ABI"
    make clean || exit 1
    make -j$CORES || exit 1
    make install || exit 1
    echo "making ffmpeg success: $ABI"

    copy_ffmpeg_header_file
    make distclean
    cd $ROOT_DIR
}

function link_one_so {
    cd $BUILD_PREFIX/lib
    echo "link one so: $ABI"
    $CROSS_PREFIX-gcc -Wl,-soname,libffmpegsz.so -shared --sysroot=$SYSROOT \
        -Wl,--whole-archive libavcodec.a libavfilter.a libavformat.a libavutil.a libswresample.a libswscale.a libx264.a\
        -Wl,--no-whole-archive -lz -lc -lm \
        $EXTRA_ARCH_LDFLAGS \
        -o libffmpegsz.so || exit 1
    echo "link one so success: $ABI"
    cd $ROOT_DIR
}

if [ "$BUILD_ARM_V7" == "1" ]; then
    ABI="armeabi-v7a"
    CROSS_PREFIX=arm-linux-androideabi
    BUILD_PREFIX="$BUILD_ROOT_DIR/$ABI"
    TOOLCHAIN="arm-linux-androideabi-$GCC_VERSION"
    MIN_PLATFORM=$MIN_PLATFORM_32
    X264_HOST="arm-linux"
    FFMPEG_ARCH_OPTIONS="--arch=arm --cpu=armv7-a --enable-neon --enable-thumb"
    EXTRA_ARCH_CFLAGS="-march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16 -mthumb"
    EXTRA_ARCH_LDFLAGS="-march=armv7-a -Wl,--fix-cortex-a8"

    cleanup
    prepare
    build_x264
    build_ffmpeg
    link_one_so
fi


if [ "$BUILD_ARM_64" == "1" ]; then
    ABI="arm64-v8a"
    CROSS_PREFIX=aarch64-linux-android
    BUILD_PREFIX="$BUILD_ROOT_DIR/$ABI"
    TOOLCHAIN="aarch64-linux-android-$GCC_VERSION"
    MIN_PLATFORM=$MIN_PLATFORM_64
    X264_HOST="aarch64-linux"
    FFMPEG_ARCH_OPTIONS="--arch=aarch64 --cpu=armv8-a"
    EXTRA_ARCH_CFLAGS="-march=armv8-a"
    EXTRA_ARCH_LDFLAGS=""

    cleanup
    prepare
    build_x264
    build_ffmpeg
    link_one_so
fi

if [ "$BUILD_X86_32" == "1" ]; then
    ABI="x86"
    CROSS_PREFIX=i686-linux-android
    BUILD_PREFIX="$BUILD_ROOT_DIR/$ABI"
    TOOLCHAIN="x86-$GCC_VERSION"
    MIN_PLATFORM=$MIN_PLATFORM_32
    X264_HOST="i686-linux"
    FFMPEG_ARCH_OPTIONS="--arch=x86 --cpu=i686"
    EXTRA_ARCH_CFLAGS="-march=i686"
    EXTRA_ARCH_LDFLAGS=""

    cleanup
    prepare
    build_x264
    build_ffmpeg
    link_one_so
fi

if [ "$BUILD_X86_64" == "1" ]; then
    ABI="x86_64"
    CROSS_PREFIX=x86_64-linux-android
    BUILD_PREFIX="$BUILD_ROOT_DIR/$ABI"
    TOOLCHAIN="x86_64-$GCC_VERSION"
    MIN_PLATFORM=$MIN_PLATFORM_64
    X264_HOST="x86_64-linux"
    FFMPEG_ARCH_OPTIONS="--arch=x86_64 --cpu=x86-64"
    EXTRA_ARCH_CFLAGS="-march=x86-64"
    EXTRA_ARCH_LDFLAGS="-Wl,-Bsymbolic"

    cleanup
    prepare
    build_x264
    build_ffmpeg
    link_one_so
fi

```

