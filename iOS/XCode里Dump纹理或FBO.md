### 1. [安装chisel ](https://github.com/facebook/chisel)

```
brew update
brew install chisel
```

if `.lldbinit` file doesn't exist you can create it & open it by tapping on the terminal（注意要创建在根目录）

```
touch ~/.lldbinit
open ~/.lldbinit
```

Then add the following line to your `~/.lldbinit` file.

```
# ~/.lldbinit
...
command script import /usr/local/opt/chisel/libexec/fbchisellldb.py
```

- Note that if you are installing on an M1 Mac, the path above should be `/opt/homebrew/opt/chisel/libexec/fbchisellldb.py` instead.

Alternatively, download chisel and add the following line to your *~/.lldbinit* file.

```
# ~/.lldbinit
...
command script import /path/to/fbchisellldb.py
```

The commands will be available the next time `Xcode` starts.



### 2. 使用步骤

// 在lldb初始化一个img临时变量，调用 SSPEditorDumpUtils 的dump接口生成UIImage
e UIImage *$img = SSPEditorDumpFramebuffer(*out_buf1);
// img变量 可以重复使用
e $img = SSPEditorDumpFramebuffer(*out_buf2);
// 调命令预览画面
visualize $img

```objective-c
#import <Foundation/Foundation.h>
#include "../../sargeras/render/sspe_texture.h"
#include "../../sargeras/render/sspe_framebuffer.h"
#include "../../sargeras/common/sspe_image.h"

NS_ASSUME_NONNULL_BEGIN

UIImage* SSPEditorDumpTexture(const ssp::sargeras::Texture& texture);

UIImage* SSPEditorDumpTexture(GLuint texture_id, uint32_t width, uint32_t height);

UIImage* SSPEditorDumpFramebuffer(ssp::sargeras::Framebuffer& framebuffer);

UIImage* SSPEditorDumpImage(const ssp::sargeras::Image& image, uint32_t target_width = 0, uint32_t target_height = 0);

@interface SSPEditorDumpUtils : NSObject

@end

NS_ASSUME_NONNULL_END
```



```objective-c
#import "SSPEditorDumpUtils.h"
#import "SSPEditorImageLoader.h"

UIImage* SSPEditorDumpTexture(const ssp::sargeras::Texture& texture) {
  auto image = texture.getImage(0, 0, texture.width(), texture.height());
  if (image && image.get()) {
    return SSPEditorDumpImage(*image.get());
  }
  return nil;
}

UIImage* SSPEditorDumpTexture(GLuint texture_id, uint32_t width, uint32_t height) {
  auto texture = std::make_shared<ssp::sargeras::Texture>(texture_id, width, height, true);
  if (texture && texture.get()) {
    return SSPEditorDumpTexture(*texture.get());
  }
  return nil;
}

UIImage* SSPEditorDumpFramebuffer(ssp::sargeras::Framebuffer& framebuffer) {
  auto image = (const_cast<ssp::sargeras::Framebuffer&>(framebuffer)).getImage(0, 0, framebuffer.width(), framebuffer.height());
  if (image && image.get()) {
    return SSPEditorDumpImage(*image.get());
  }
  return nil;
}

UIImage* SSPEditorDumpImage(const ssp::sargeras::Image& image, uint32_t target_width /*= 0*/, uint32_t target_height /*= 0*/) {
  return ssp::sargeras::ImageLoaderIOS::loadNativeImage(image, target_width, target_height);
}

@implementation SSPEditorDumpUtils

@end
```



### 3.其他

1. visualize 干了啥？
   会将UIImage保存到/tmp/xcode_debug_images目录下，然后用系统预览打开
   清理调试图片可以在 访达 cmd+shift+g 访问 /tmp/xcode_debug_images，清理该文件夹即可
2. 嫌visualize 太长, 在.lldbinit 文件末尾添加重命名指令
   command alias view visualize