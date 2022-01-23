https://blog.csdn.net/aa20274270/article/details/83658772



### MaxOS

安装 软件

1. xcode-select --install

2. 安装 Cmake ：

https://blog.csdn.net/qq_21046135/article/details/78767134

 

安装 ninja

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

 

运行例子

1.  samples/lightbulb samples/assets/models/monkey/monkey.obj
————————————————
版权声明：本文为CSDN博主「aa20274270」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/aa20274270/article/details/83658772



### IOS （xcode上跑filament）

1.  如果是真机的话，直接执行

./build.sh -p ios debug

 

2. 如果是要模拟器的话，直接执行

./build.sh -s -p ios debug

 

3. 在目录 filsment/ios/sample/hello-triangle/hello-triangle/ 打开xcodeproj，就可以在直接在 xcode中 跑一个三角形例子，

可以参考：https://github.com/google/filament/blob/master/ios/samples/README.md
————————————————
版权声明：本文为CSDN博主「aa20274270」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/aa20274270/article/details/83658772