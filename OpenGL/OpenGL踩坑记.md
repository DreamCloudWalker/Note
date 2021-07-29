### VBO

* GLES20.glBufferData或GLES20.glBufferSubData中传入的Buffer类型必须是转换后的类型。之前一个写法，由于kotlin有类型推断，不会出错。如下：

  ```kotlin
  val verticesVBOSize = vertexCnt * BYTES_PER_VERTEX
  val vbb = ByteBuffer.allocateDirect(verticesVBOSize)
  vbb.order(ByteOrder.nativeOrder()).asFloatBuffer().put(verticesData).position(0)
  
  val uvVBOSize = UV_DATA.size * BYTES_PRE_FLOAT
  val ubb = ByteBuffer.allocateDirect(uvVBOSize)
  ubb.order(ByteOrder.nativeOrder()).asFloatBuffer().put(UV_DATA).position(0)
  ```

  但在Java中这么写就会出错，且难以调试，没有glError，只会绘制不出来。

  ```Java
  int vertexVboSize = mVerticesData.length * BYTES_PRE_FLOAT;
  ByteBuffer vertexBuffer = ByteBuffer.allocateDirect(vertexVboSize);
  vertexBuffer.order(ByteOrder.nativeOrder()).asFloatBuffer().put(mVerticesData).position(0);
  
  int uvVboSize = mUVData.length * BYTES_PRE_FLOAT;
  ByteBuffer uvBuffer = ByteBuffer.allocateDirect(uvVboSize);
  vertexBuffer.order(ByteOrder.nativeOrder()).asFloatBuffer().put(mUVData).position(0);
  ```

  改正后的Java写法如下：

  ```java
  int vertexVboSize = mVerticesData.length * BYTES_PRE_FLOAT;
  FloatBuffer vertexBuffer = ByteBuffer.allocateDirect(vertexVboSize).order(ByteOrder.nativeOrder()).asFloatBuffer();
  vertexBuffer.put(mVerticesData).position(0);
  
  int uvVboSize = mUVData.length * BYTES_PRE_FLOAT;
  FloatBuffer uvBuffer = ByteBuffer.allocateDirect(uvVboSize).order(ByteOrder.nativeOrder()).asFloatBuffer();
  uvBuffer.put(mUVData).position(0);
  
  // update buffer data
  GLES20.glBufferData(
    GLES20.GL_ARRAY_BUFFER,
    vertexVboSize + uvVboSize,
    null,
    GLES20.GL_STATIC_DRAW
  );
  OpenglUtils.checkGlError("vbo glBufferData");
  GLES20.glBufferSubData(GLES20.GL_ARRAY_BUFFER, 0, vertexVboSize, vertexBuffer);
  GLES20.glBufferSubData(GLES20.GL_ARRAY_BUFFER, vertexVboSize, uvVboSize, uvBuffer);
  OpenglUtils.checkGlError("vbo glBufferSubData");
  GLES20.glBindBuffer(GLES20.GL_ARRAY_BUFFER, 0); // recover
  OpenglUtils.checkGlError("vbo glBindBuffer");
  ```

  



### FBO

* FBO的buffer解绑时glBindFramebuffer的参数是0，不能用-1，不然会出现1286错误。VBO Buffer解绑参数有待确认。

* 如果前面代码有裁剪测试（GL_SISSOR_TEST）,需要先关掉；

* 用glFrameBufferTexture2D传入的纹理ID，需预先生成传入一个new int[1],然后glGenTexture，

  并且GLES20.glTexImage2D(GLES20.GL_TEXTURE_2D, 0, GLES20.GL_RGBA, width, height, 0,GLES20.GL_RGBA, GLES20.GL_UNSIGNED_BYTE, null);

  不然后面glCheckFramebufferStatus(GLES20.GL_FRAMEBUFFER)时会报GLES20.GL_FRAMEBUFFER_INCOMPLETE_ATTACHMENT错误。

  或者如果glTexImage2D用的不对，也会出现GL_FRAMEBUFFER_INCOMPLETE_ATTACHMENT。比如不能绑***单通道的图片***，只能GL_RGBA。

