# Android 上的 Vulkan 着色器编译器

Vulkan 应用管理着色器的方式必须不同于 OpenGL ES 应用：在 OpenGL ES 中，您提供的着色器应是构成 GLSL 着色器程序源文本的一组字符串，而 Vulkan API 则要求着色器的形式为 [SPIR-V](https://www.khronos.org/spir) 模块中的入口点。

[NDK 版本 12 及更高版本](https://github.com/android-ndk/ndk/wiki)包括一个用于将 GLSL 编译到 SPIR-V 中的运行时库。该运行时库与 [Shaderc](https://github.com/google/shaderc) 开源项目中的相同，并使用相同的 [Glslang GLSL](https://github.com/KhronosGroup/glslang) 参考编译器作为其后端。默认情况下，Shaderc 版本的编译器会假设您在针对 Vulkan 进行编译。检查您的代码是否对 Vulkan 有效后，编译器会自动启用 `KHR_vulkan_glsl` 扩展程序。Shaderc 版本的编译器还会生成与 Vulkan 兼容的 SPIR-V 代码。

您可以选择在开发期间将 SPIR-V 模块编译到您的 Vulkan 应用中，这种做法称为提前（即 AOT）编译。此外，您也可以让您的应用在运行时视需要将设备搭载的或按一定程序生成的着色器源代码编译成这些模块。这种做法称为运行时编译。Android Studio 已集成对构建 Vulkan 着色器的支持。

本页面的其余部分将详细介绍每种做法，然后说明如何将着色器编译集成到您的 Vulkan 应用中。



## AOT 编译

如下面的部分所述，您可以通过两种方式实现着色器 AOT 编译。



#### 使用 Android Studio

将着色器放入 `app/src/main/shaders/` 后，Android Studio 会通过[文件扩展名](https://github.com/google/shaderc/tree/main/glslc)识别着色器，

Table 1. Shader Stage Selection

| Shader Stage           | Shader File Extension | `<stage>`     |
| ---------------------- | --------------------- | ------------- |
| vertex                 | `.vert`               | `vertex`      |
| fragment               | `.frag`               | `fragment`    |
| tesselation control    | `.tesc`               | `tesscontrol` |
| tesselation evaluation | `.tese`               | `tesseval`    |
| geometry               | `.geom`               | `geometry`    |
| compute                | `.comp`               | `compute`     |

并完成以下操作：

- 在该目录下以递归方式编译所有着色器文件。
- 将 .spv 后缀附加到编译成的 SPIR-V 着色器文件。
- 将 SPIRV 着色器打包到 APK 的 `assets/shaders/` 目录中。

应用会在运行时从对应的 `assets/shaders/` 位置加载已编译的着色器；编译的 spv 着色器文件结构与 `app/src/main/shaders/` 下应用的 GLSL 着色器文件结构相同：

```cpp
AAsset* file = AAssetManager_open(assetManager,
                     "shaders/tri.vert.spv", AASSET_MODE_BUFFER);
size_t fileLength = AAsset_getLength(file);
char* fileContent = new char[fileLength];
AAsset_read(file, fileContent, fileLength);
```

如下例所示，您可以在 Gradle DSL `shaders` 块内部配置 Shaderc 编译标志：

[Groovy](https://developer.android.com/ndk/guides/graphics/shader-compilers?hl=zh-cn#groovy)[Kotlin](https://developer.android.com/ndk/guides/graphics/shader-compilers?hl=zh-cn#kotlin)

```groovy
android {
  defaultConfig {
    shaders {
      glslcArgs.addAll(['-c', '-g'])
      scopedArgs.create('lights') {
        glslcArgs.addAll(['-DLIGHT1=1', '-DLIGHT2=0'])
      }
    }
  }
}
```

`glslcArgs` 适用于所有着色器编译；`scopedArgs` 仅在针对该作用域进行编译时才适用。以上示例创建的作用域参数 `lights` 仅适用于 `app/src/main/shaders/lights/` 目录下的 GLSL 着色器。要获取可用编译标记的完整列表，请参阅 [glslc](https://github.com/google/shaderc/tree/main/glslc)。请注意，NDK 中的 Shaderc 是 NDK 发布时该 GitHub 代码库的快照；如下一部分所述，您可以使用命令 `glslc --help` 获取该版本支持的确切标记。



#### 离线命令行编译

您可以使用 glslc 命令行编译器将 GLSL 着色器编译到独立于主应用的 SPIR-V 中。为了支持这种使用模式，NDK 版本 12 及更高版本会将某个版本的预构建 glslc 和相关工具封装到 `<android-ndk-dir>/shader-tools/` 目录中。

您也可以从 [Shaderc](https://github.com/google/shaderc) 项目获得编译器；请按照相关说明构建二进制版本。

glslc 为着色器编译提供了丰富的[命令行选项](https://github.com/google/shaderc/tree/main/glslc)，以满足应用的各种要求。

glslc 工具可以将单个源文件编译到具有单个着色器入口点的 SPIR-V 模块中。默认情况下，输出文件的名称与源文件的名称相同，但附加了 `.spv` 扩展名。

使用文件扩展名可以告知 glslc 工具要编译的图形着色器阶段，或者指示某个计算着色器是否正在编译中。要了解如何使用这些文件扩展名，以及可与该工具搭配使用的选项，请参阅 [glslc](https://github.com/google/shaderc/tree/main/glslc) 手册中的[着色器阶段规范](https://github.com/google/shaderc/tree/main/glslc#user-content-shader-stage-specification)。



## 运行时编译

对于运行时着色器的 JIT 编译，NDK 会提供同时拥有 C API 和 C++ API 的 libshaderc 库。

C++ 应用应当使用 C++ API。我们建议采用其他语言编写的应用使用 C API，因为 C ABI 的抽象层级更低，可以提供更好的稳定性。

下面的示例显示了如何使用 C++ API：

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <shaderc/shaderc.hpp>

std::vector<uint32_t> compile_file(const std::string& name,
                                   shaderc_shader_kind kind,
                                   const std::string& data) {
  shaderc::Compiler compiler;
  shaderc::CompileOptions options;

  // Like -DMY_DEFINE=1
  options.AddMacroDefinition("MY_DEFINE", "1");

  shaderc::SpvCompilationResult module = compiler.CompileGlslToSpv(
      data.c_str(), data.size(), kind, name.c_str(), options);

  if (module.GetCompilationStatus() !=
      shaderc_compilation_status_success) {
    std::cerr << module.GetErrorMessage();
  }

  std::vector<uint32_t> result(module.cbegin(), module.cend());
  return result;
}
```



## 集成到您的项目中

您可以使用项目的 `Android.mk` 文件或 Gradle 将 Vulkan 着色器编译器集成到您的应用中。



#### Android.mk

执行以下步骤，使用项目的 `Android.mk` 文件集成着色器编译器。

1. 在 Android.mk 文件中添加以下这几行代码：

   ```shell
   include $(CLEAR_VARS)
        ...
   LOCAL_STATIC_LIBRARIES := shaderc
        ...
   include $(BUILD_SHARED_LIBRARY)
   
   $(call import-module, third_party/shaderc)
   ```

2. 在应用的 Application.mk 中将 APP_STL 设置为 `c++_static`、`c++_shared`、`gnustl_static` 或 `gnustl_shared` 之一

   

#### Gradle 的 CMake 集成

1. 在终端窗口中，导航至 `ndk_root/sources/third_party/shaderc/`。

2. 运行以下命令以构建 NDK 的 Shaderc。在您使用的每个 NDK 版本上，您只需运行此命令一次：

   ```shell
   $ ../../../ndk-build NDK_PROJECT_PATH=. APP_BUILD_SCRIPT=Android.mk \
   APP_STL:=<stl_version> APP_ABI=all libshaderc_combined
   ```

   此命令会将两个文件夹置于 <ndk_root>/sources/third_party/shaderc/ 中。目录结构如下：

   ```shell
   include/
     shaderc/
       shaderc.h
       shaderc.hpp
   libs/
     <stl_version>/
       {all of the abis}
          libshaderc.a
   ```

3. 按照您通常对[外部库](https://github.com/android/ndk-samples/blob/master/hello-libs/app/src/main/cpp/CMakeLists.txt)执行的操作，使用 [`target_include_directories`](https://cmake.org/cmake/help/v3.12/command/target_include_directories.html?highlight=target_include_directories) 和 [`target_link_libraries`](https://cmake.org/cmake/help/v3.12/command/target_link_libraries.html?highlight=target_link_lib#target-link-libraries) 添加生成的 include 和 lib。您应用的 STL 类型必须与 `stl_version` 中指定的 `stl` 类型之一匹配。NDK 建议使用 `c++_shared` 或 `c++_static`，但也支持 `gnustl_static` 和 `gnustl_shared`。



## 获取最新的 Shaderc

NDK 中的 Shaderc 来自[ Android 源代码树](https://android.googlesource.com/platform/external/shaderc)，它是[上游 Shaderc 代码库](https://github.com/google/shaderc)的快照。如果您需要最新的 Shaderc，请参阅[构建说明](https://github.com/google/shaderc/blob/master/README.md)获取详细信息。总体步骤如下：

1. 下载最新的 Shaderc：

   ```shell
   git clone https://github.com/google/shaderc.git
   ```

2. 更新依赖项：

   ```shell
   ./utils/git-sync-dep
   ```

3. 编译 Shaderc：

   ```shell
   <ndk_dir>/ndk-build NDK_PROJECT_PATH=. APP_BUILD_SCRIPT=Android.mk \
       APP_STL:=c++_static APP_ABI=all libshaderc_combined -j16
   ```

4. 将您的项目配置为在构建脚本文件中使用自己的 Shaderc 版本。