颜色缓冲、深度缓冲、模板缓冲等结合称为帧缓冲，存放在 gpu 内存中。

创建窗口时通常会自动生成和配置默认帧缓冲，只有默认帧缓冲会渲染在屏幕上，因此其他帧缓冲也称为离屏渲染。

`glGenFramebuffer(count, &fbo)`，创建帧缓冲对象。

`glBindFramebuffer(GL_FRAMEBUFFER, fbo)`，绑定帧缓冲对象。

> 绑定后，所有读取和写入帧缓冲的操作都会影响当前绑定的帧缓冲，也可以使用 `GL_READ_FRAMEBUFFER` 或 `GL_DRAW_FRAMEBUFFER` 分别绑定读取和写入的帧缓冲。

一个完整的帧缓冲需要满足以下条件：

- 附加至少一个缓冲，且至少一个颜色缓冲。

- 所有附件都必须完整（拥有内存）。

- 每个缓冲都有相同的样本数。

> 如果 `glCheckFramebufferStatus(GL_FRAMEBUFFER) == GL_FRAMEBUFFER_COMPLETE` 满足，则帧缓冲完整。

之后的渲染会将结果存放到绑定的附件中。

`glDeleteFrameBuffers(count, &fbo)`，删除缓冲对象。

## 附件

附件是一个内存地址，作为帧缓冲的缓冲，可以创建纹理缓冲或渲染缓冲。

### 纹理附件

创建纹理附件和创建普通纹理差不多，主要区别是只需要为纹理分配内存而不需要用具体数据填充，且通常纹理大小和视窗大小一致。

```cpp
GLuint texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);


glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, 800, 600, 0, GL_RGB, GL_UNSIGNED_BYTE, nullptr);

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

附加纹理对象：`glFramebufferTexture2D(GLenum target, GLenum attachment, GLenum textarget, GLuint texture, GLint level);`

- target: 帧缓冲目标。

- attachment: 附件类型。

- textarget: 纹理类型。

- level: 多级渐远纹理级别。

> 如 `glFrameBufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0)`。

如果要附加深度缓冲，则纹理的格式类型需要设置为 `GL_DEPTH_COMPONENTXX` 表示深度缓冲的大小。

```cpp
glTexImage2D(GL_TEXTURE_2D, 0, GL_DEPTH_COMPONENT24, 800, 600, 0, GL_DEPTH_COMPONENT24, GL_FLOAT, nullptr);
```

### 渲染缓冲对象附件

渲染缓冲对象是在纹理之后，专门引入的用作缓冲帧附件使用，它的数据将存储为 opengl 原生的渲染格式，不会做任何针对纹理格式的转换。

渲染缓冲对象通常是只写的（用户代码不需要访问具体的数据是什么），因此通常用于深度和模板附件。

> 虽然依然可以通过 `glReadPixels()` 读取特定区域的像素。

```cpp
GLuint rbo;
glGenRenderbuffers(1, &rbo);

glBindRenderbuffer(GL_RENDERBUFFER, rbo);

glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH_COMPONENT24, 800, 600);

glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_RENDERBUFFER, rbo);
```

## 后处理

通过将整个场景渲染到一个纹理图像上，可以在输出的片段着色器中实现一些特殊的效果，称为后处理。
