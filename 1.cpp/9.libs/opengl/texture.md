纹理通常是一张 2D 图片。

> 除了图片之外，纹理也可以用来存储大量的数据，发送到着色器上。

使用纹理坐标获取纹理颜色的过程叫做采样(sampling)。纹理坐标在 x 和 y 轴上，以左下角为原点，范围为 0~1。

## 纹理环绕

纹理环绕方式是当纹理坐标范围超过 `(1,1)` 后 opengl 的绘制行为。

- `GL_REPEAT`，重复纹理图像，默认行为。

- `GL_MIRRORED_REPEAT`，镜像重复纹理图像。

- `GL_CLAMP_TO_EDGE`，超出部分为纹理边缘颜色。

- `GL_CLAMP_TO_BORDER`，超出颜色为指定边缘颜色。

![纹理环绕示例](https://learnopengl-cn.github.io/img/01/06/texture_wrapping.png)

纹理环绕方式通过 `glTexParameterr*` 函数簇对每一个轴单独设置：

```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
```

- `GL_TEXTURE_2D`，指定为 2D 纹理。

- `GL_TEXTURE_WRAP_S`、`GL_TEXTURE_WRAP_T`，纹理环绕选项，s 和 t 轴。

如果使用 `GL_CLAMP_TO_BORDER`，还需额外设置边缘颜色。

```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_BORDER);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_BORDER);

auto color = std::array{1.f, 1.f, 1.f, 1.f};
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, color.data());
```

## 纹理过滤

纹理坐标不依赖于分辨率，因此当纹理分辨率尺寸和物体尺寸不一致时，就需要进行纹理过滤以将纹理映射到物体上。

- `GL_NEAREST`，邻近过滤。选择离中心点最近的像素作为样本颜色。

- `GL_LINEAR`，线性过滤。基于纹理坐标附近的像素进行线性插值计算得到样本颜色。

将低分辨率纹理映射到大物体上时，如果选择 `GL_NEAREST`，会得到带颗粒状的图案，如果选择 `GL_LINEAR`，则会得到更模糊但平滑的图案。

![邻近过滤和线性过滤比较](https://learnopengl-cn.github.io/img/01/06/texture_filtering.png)

通过 `glTexParameter*` 函数簇分别设置放大缩小时的纹理过滤方式。

```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

## 多级渐远纹理

多级渐远纹理(mipmap)：当物体处于较远位置时，使用更低像素的纹理，以提高性能。

opengl 提供了 `glGenerateMipmap()` 函数，创建纹理之后可以让 opengl 自动生成对应的 mipmap。

进行 mimap 时，会先计算 LOD(level of detail)值，然后选择合适的 mimap 层级。在选择层级时，同样需要设置层级选择的方式：

- nearest，邻近层级选择。

- linear，线性插值混合层级。

同样通过 `glTexParameterr*` 函数簇进行设置：

```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST_MIPMAP_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_NEAREST);
```

- `GL_NEAREST_MIPMAP_NEAREST`，使用邻近进行纹理过滤和层级选择。

- `GL_LINEAR_MIPMAP_NEAREST`，使用线性插值进行纹理过滤，使用邻近进行层级选择。

> PS：只有纹理缩小才能设置多级渐远纹理层级选择的方式。

## 加载和创建纹理

需要将纹理图像进行解码，然后才能传递给 opengl 使用。

- `glGenTextures()`，创建纹理对象。

- `glBindTexture()`，绑定纹理对象。

- `glTexImage2D(target, level, i_format, w, h, border, format, type, data)`，2D 纹理操作。

  - `target`，操作。通常为 `GL_TEXTURE_2D`，表示定义一个 2D 纹理。

  - `level`，指定 LOD 等级，基本等级为 0。

  - `i_format`，指定纹理内部格式。

  - `w`、`h`，宽高。

  - `border`，遗留问题，必须是 0。

  - `format`，传入数据的格式。

  - `type`、`data`，单个数据类型和数据指针。

glsl 内置了一个提供给纹理对象使用的数据类型，称为采样器，其以纹理类型作为后缀，如 `sampler1D`、`sampler2D`。使用内置函数 `texture()` 采样纹理颜色，第一个参数是采样器，第二个参数是对应的纹理坐标，输出的是颜色。

#### 纹理单元

采样器变量通过 `uniform` 声明，一个采样器变量实际上引用一个纹理单元，默认情况下采样器变量对应的是纹理单元 0。因此，如果只需要一个纹理时，可以不需要通过 `glUniform1i()` 为其赋值。

在绑定纹理对象之前，通常还需要通过 `glActiveTexture()` 激活对应的纹理单元，默认会激活纹理单元 0。opengl 保证至少由 16 个纹理单元可以使用，从 `GL_TEXTURE0` 到 `GL_TEXTURE15`。

#### 示例

```cpp
class GLWidget : public QOpenGLWindow, protected QOpenGLExtraFunctions {
public:
  auto initializeGL() -> void override {
    initializeOpenGLFunctions();
    glClearColor(1, 1, 1, 1);
    compile_shader();
    glUseProgram(pro_);

    auto vertices = std::array{
        0.5f,  0.5f,  0.0f, 1.0f, 1.0f, // 右上
        0.5f,  -0.5f, 0.0f, 1.0f, 0.0f, // 右下
        -0.5f, -0.5f, 0.0f, 0.0f, 0.0f, // 左下
        -0.5f, 0.5f,  0.0f, 0.0f, 1.0f  // 左上
    };

    auto indices = std::array{
        0, 1, 3, //
        1, 2, 3  //
    };

    glGenVertexArrays(1, &vao_);
    glBindVertexArray(vao_);

    auto vbo = GLuint{};
    glGenBuffers(1, &vbo);
    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    glBufferData(GL_ARRAY_BUFFER, vertices.size() * sizeof(float), vertices.data(), GL_STATIC_DRAW);

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 5 * sizeof(float), (void *)0);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, 5 * sizeof(float), (void *)(3 * sizeof(float)));
    glEnableVertexAttribArray(1);

    auto ebo = GLuint{};
    glGenBuffers(1, &ebo);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, indices.size() * sizeof(int), indices.data(), GL_STATIC_DRAW);

    // 纹理
    auto img = QImage{"./container.jpg"}.convertToFormat(QImage::Format_RGB888);
    glGenTextures(1, &tex_);
    glBindTexture(GL_TEXTURE_2D, tex_);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, img.width(), img.height(), 0, GL_RGB, GL_UNSIGNED_BYTE, img.bits());

    glBindVertexArray(0);
    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0);
  }

  auto paintGL() -> void override {
    glClear(GL_COLOR_BUFFER_BIT);
    glBindVertexArray(vao_);
    glBindTexture(GL_TEXTURE_2D, tex_);
    glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
  }

  auto resizeGL(int w, int h) -> void override { glViewport(0, 0, w, h); }

  auto compile_shader() -> void {
    auto v_shader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(v_shader, 1, &v_shader_s, nullptr);
    glCompileShader(v_shader);
    check_compile(v_shader);

    auto f_shader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(f_shader, 1, &f_shader_s, nullptr);
    glCompileShader(f_shader);
    check_compile(f_shader);

    pro_ = glCreateProgram();
    glAttachShader(pro_, v_shader);
    glAttachShader(pro_, f_shader);
    glLinkProgram(pro_);
    check_link(pro_);

    glDeleteShader(v_shader);
    glDeleteShader(f_shader);
  }

  auto check_compile(GLuint shader) -> void {
    auto suc = 0;
    glGetShaderiv(shader, GL_COMPILE_STATUS, &suc);
    if (!suc) {
      char buf[1024];
      glGetShaderInfoLog(shader, sizeof(buf), nullptr, buf);
      std::println("compile failed: {}", buf);
    }
  }

  auto check_link(GLuint program) -> void {
    auto suc = 0;
    glGetProgramiv(program, GL_LINK_STATUS, &suc);
    if (!suc) {
      char buf[1024];
      glGetProgramInfoLog(program, sizeof(buf), nullptr, buf);
      std::println("link failed: {}", buf);
    }
  }

private:
  GLuint vao_;
  GLuint pro_;
  GLuint tex_;

private:
  inline static auto v_shader_s = R"(#version 330 core
    layout (location=0) in vec3 pos;
    layout (location=1) in vec2 tex_coord;
    out vec2 v_tex_coord;
    void main(){
      gl_Position = vec4(pos, 1);
      v_tex_coord = tex_coord;
    })";

  inline static auto f_shader_s = R"(#version 330 core
    out vec4 color;
    in vec2 v_tex_coord;
    uniform sampler2D tex;
    void main(){
      color = texture(tex, v_tex_coord);
    })";
};
```
