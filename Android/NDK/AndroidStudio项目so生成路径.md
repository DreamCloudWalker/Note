app目录下是build/intermediates/merged_native_libs/debug[release]/out/lib/armxxx

另外其他目录应该也有拷贝。参考The .so files are also available in the stripped_native_libs and cmake folder.



经AS编译后，生成so如下：

```shell
./app/build/intermediates/cmake/debug/obj/arm64-v8a/libarc_sateis.so
./app/build/intermediates/cmake/debug/obj/armeabi-v7a/libarc_sateis.so
./app/build/intermediates/merged_native_libs/debug/out/lib/arm64-v8a/libtest.so
./app/build/intermediates/merged_native_libs/debug/out/lib/armeabi-v7a/libtest.so
./app/build/intermediates/stripped_native_libs/debug/out/lib/arm64-v8a/libtest.so
./app/build/intermediates/stripped_native_libs/debug/out/lib/armeabi-v7a/libtest.so
```

其中obj是中间生成so，用的话可以用merged_native_libs和stripped_native_libs下的；
 区别：直接看大小就明白，stripped下203KB， merged是1129KB，应该是merged下包含一些map信息、地址等，调试用比较合适；stripped移除了这部分，release比较合适。