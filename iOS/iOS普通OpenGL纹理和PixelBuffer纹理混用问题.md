iOS创建FBO操作举例如下：

```c++
bool Framebuffer::init(uint32_t width, uint32_t height, bool highp_attachment /*= false*/) {
  if ((width != width_ || height != height_) || (id_ == 0) || isHighpColorAttachment() != highp_attachment) {
    texture_ = nullptr;
    deleteFbo();

    width_ = width;
    height_ = height;

    glGenFramebuffers(1, &id_);
    if (!id_) {
      return false;
    }
#ifdef SSPE_IOS
    if (SupportFastTextureUpload()) {
      CVPixelBufferRef pixel_buffer = CreatePixelBuffer(width, height, highp_attachment);
      if (pixel_buffer) {
        auto shared_pixel_buffer = std::shared_ptr<void>(pixel_buffer, [](void* ptr) {
          CVPixelBufferRelease((CVPixelBufferRef)ptr);
        });
        texture_ = std::make_shared<Texture>(shared_pixel_buffer, width, height, kVideoFrameFormatPixelBufferBGRA, 0);
      }
      if (!texture_) {
        Texture::Descriptor descriptor = Texture::Descriptor(highp_attachment);
        texture_ = std::make_shared<Texture>(width, height, nullptr, &descriptor);
      }
    } else {
      Texture::Descriptor descriptor = Texture::Descriptor(highp_attachment);
      texture_ = std::make_shared<Texture>(width, height, nullptr, &descriptor);
    }
#else
    texture_ = std::make_shared<Texture>(width, height);
#endif
    if (!texture_) {
      deleteFbo();
      return false;
    }

    glBindFramebuffer(GL_FRAMEBUFFER, id_);
    glBindTexture(GL_TEXTURE_2D, texture_->id());
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture_->id(), 0);
    GLenum status = glCheckFramebufferStatus(GL_FRAMEBUFFER);
    if (status != GL_FRAMEBUFFER_COMPLETE) {
      unbind();
      texture_ = nullptr;
      deleteFbo();
      SLOGE("glError: 0x%x --> %s", status, errorInfo(status).c_str());
      return false;
    }
    glBindTexture(GL_TEXTURE_2D, 0);
    glBindFramebuffer(GL_FRAMEBUFFER, 0);
    bind_thread_ids_.push_back(std::this_thread::get_id());
  }
  return true;
}

std::unique_ptr<Image> Framebuffer::getImage(uint32_t x, uint32_t y, uint32_t width, uint32_t height) {
#ifdef SSPE_IOS
  uint8_t* data = nullptr;
  int stride = 0;
  if (texture_ && texture_->pixelBuffer() && 0 == x && 0 == y && width == width_ && height == height_) {
    data = texture_->pixelBufferData(&stride);
  }
  if (data && stride > 0) {
    std::unique_ptr<Image> image(new (std::nothrow) Image(width, height, Image::Format::BGRA, stride, data));
    return image;
  } else {
    GLint pre_fbo = 0;
    glGetIntegerv(GL_FRAMEBUFFER_BINDING, &pre_fbo);

    glBindFramebuffer(GL_FRAMEBUFFER, id_);
    checkBindStatus();
    glPixelStorei(GL_PACK_ALIGNMENT, 1);
    std::unique_ptr<Image> image(new (std::nothrow) Image(width, height, Image::Format::RGBA));
    if (image && image.get() && image->data()) {
      glReadPixels(x, y, width, height, GL_RGBA, GL_UNSIGNED_BYTE, image->data());
    }

    glBindFramebuffer(GL_FRAMEBUFFER, pre_fbo);
    return image;
  }
#else
  GLint pre_fbo = 0;
  glGetIntegerv(GL_FRAMEBUFFER_BINDING, &pre_fbo);

  glBindFramebuffer(GL_FRAMEBUFFER, id_);
  checkBindStatus();
  glPixelStorei(GL_PACK_ALIGNMENT, 1);
  std::unique_ptr<Image> image(new (std::nothrow) Image(width, height, Image::Format::RGBA));
  if (image && image.get() && image->data()) {
    glReadPixels(x, y, width, height, GL_RGBA, GL_UNSIGNED_BYTE, image->data());
  }

  glBindFramebuffer(GL_FRAMEBUFFER, pre_fbo);
  return image;
#endif
}
```

如果工程中所有的都开启了SupportFastTextureUpload()，都用的PixelBuffer，那没有问题。

但如果存在混用，比如接入了别的渲染sdk，那边用的普通OpenGL纹理，则可能存在用PixelBuffer进行getImage来dump到内存的时候，会出现帧之后问题，需要加glFinish。

比如：

创建一个PixelBuffer的纹理，把这个纹理ID传给其他渲染SDK去渲染，然后用这个封装的getImage把这个纹理ID dump出来，会发现，如果不加glFinish，会发现dump不到渲染sdk绘制的东西。可能要滞后一帧后才能dump到结果。