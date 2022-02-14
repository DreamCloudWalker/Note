https://blog.csdn.net/aa20274270/article/details/83658772

https://github.com/google/filament/issues/2876



### MaxOS

安装 软件

1. xcode-select --install

2. 安装 Cmake ：

https://blog.csdn.net/qq_21046135/article/details/78767134

 

安装 ninja（）

1. 安装 re2c 

http://macappstore.org/re2c/

2. 安装 graphviz

brew install graphviz

3. 安装 ninja

http://macappstore.org/ninja/

 

4. 执行：

mkdir out

cd out

mkdir cmake-release

cd cmake-release

 

cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=../release/filament ../..

ninja



// 或增加一些配置，比如Metal开关。The `CMAKE_OSX_DEPLOYMENT_TARGET` flag can be used to force an older SDK version.

cmake -G Ninja -DCMAKE_BUILD_TYPE=Release <font color="red">-DFILAMENT_SUPPORTS_METAL:BOOL=OFF</font> -DCMAKE_OSX_DEPLOYMENT_TARGET=10.13 -DCMAKE_INSTALL_PREFIX=../release/filament ../..

ninja

 

运行例子

1.  samples/lightbulb samples/assets/models/monkey/monkey.obj



### IOS （xcode上跑filament）

1.  如果是真机的话，直接执行

./build.sh -p ios debug

 

2. 如果是要模拟器的话，直接执行

./build.sh -s -p ios debug

 

3. 在目录 filsment/ios/sample/hello-triangle/hello-triangle/ 打开xcodeproj，就可以在直接在 xcode中 跑一个三角形例子，

可以参考：https://github.com/google/filament/blob/master/ios/samples/README.md