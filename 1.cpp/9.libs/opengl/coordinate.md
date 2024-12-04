## 坐标变换

通常来说，一个物体的顶点坐标初始为局部坐标，然后依次转换到世界坐标、观察坐标、裁剪坐标，并最终以屏幕坐标结束。

- 局部坐标 -> 世界坐标的变换矩阵称为模型矩阵(model)。

- 世界坐标 -> 观察坐标的变换矩阵称为观察矩阵(view)。

- 观察坐标 -> 裁剪坐标的变换矩阵称为投影矩阵(projection)。裁剪坐标会被处理到 -1~1 的范围。

- 裁剪坐标 -> 屏幕坐标的过程称为视口变换(viewport)。视口变换将坐标从 -1~1 转换到 `glViewport()` 定义的坐标范围内。此过程由 opengl 自动进行。

将局部坐标转换到裁剪坐标的变换公式为：

$$
V_{clip} = M_{projection} \cdot M_{view} \cdot M_{model} \cdot V_{local}
$$

由投影矩阵构造的观察箱也被称为平截头体(Frustum)，在平截头体范围内的坐标会被输出到屏幕上。

当坐标被转换到裁剪坐标之后，会进行透视除法，即将 4D 裁剪空间坐标转换到 3D 标准化设备坐标的过程，将 xyz 分量分别除以 w 分量。这一步会在顶点着色器运行的最后自动进行。

## 投影

#### 正射投影

正射投影定义一个立方体的平截头体，一个正射投影由宽、高、近平面和远平面构成。

可以使用 Qt 提供的 `mat.ortho()` 函数创建正射投影矩阵。

```cpp
auto mat = QMatrix4x4{};
mat.ortho(left, right, bottom, top, near, far);
```

#### 透视投影

透视矩阵会修改顶点坐标的 w 分量，使得离观察坐标越远的顶点坐标 w 分量越大，从而得到进大远小的效果。

变换得到的裁剪坐标都会在 -w~w 范围内。

可以使用 Qt 提供的 `mat.perspective()` 函数创建透视投影矩阵。

```cpp
auto mat = QMatrix4x4{};
mat.perspective(fov, ratio, near, far);
```

- fov，视野的垂直可见范围。

- ratio，输出的宽高比。

## 右手坐标系

opengl 的惯例坐标系为右手坐标系，即正 x 轴沿右手方向，y 轴沿正上方，z 轴正方形穿出屏幕。

## z 缓冲

OpenGL 的深度信息存放在 z 缓冲中，也称为深度缓冲。当开启深度测试后，片段输出颜色时，会将其深度值和 z 缓冲深度值进行判断，如果片段处于其他片段之后，会被丢弃，否则覆盖。

默认深度测试是关闭的，需要通过 `glEnable()` 开启。

```cpp
glEnable(GL_DEPTH_TEST);
```

开启深度测试后，每次渲染之前还需要清空深度缓冲区。

```cpp
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

## 示例

```cpp
class GLWidget : public QOpenGLWindow, protected QOpenGLExtraFunctions {
public:
  auto initializeGL() -> void override {
    initializeOpenGLFunctions();
    glClearColor(1, 1, 1, 1);
    compile_shader();
    glUseProgram(pro_);
    glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);

    // 12个三角形
    auto vertices = std::array{
        -0.5f, -0.5f, -0.5f, 0.5f,  -0.5f, -0.5f, 0.5f,  0.5f,  -0.5f,
        0.5f,  0.5f,  -0.5f, -0.5f, 0.5f,  -0.5f, -0.5f, -0.5f, -0.5f, //

        -0.5f, -0.5f, 0.5f,  0.5f,  -0.5f, 0.5f,  0.5f,  0.5f,  0.5f,
        0.5f,  0.5f,  0.5f,  -0.5f, 0.5f,  0.5f,  -0.5f, -0.5f, 0.5f, //

        -0.5f, 0.5f,  0.5f,  -0.5f, 0.5f,  -0.5f, -0.5f, -0.5f, -0.5f,
        -0.5f, -0.5f, -0.5f, -0.5f, -0.5f, 0.5f,  -0.5f, 0.5f,  0.5f, //

        0.5f,  0.5f,  0.5f,  0.5f,  0.5f,  -0.5f, 0.5f,  -0.5f, -0.5f,
        0.5f,  -0.5f, -0.5f, 0.5f,  -0.5f, 0.5f,  0.5f,  0.5f,  0.5f, //

        -0.5f, -0.5f, -0.5f, 0.5f,  -0.5f, -0.5f, 0.5f,  -0.5f, 0.5f,
        0.5f,  -0.5f, 0.5f,  -0.5f, -0.5f, 0.5f,  -0.5f, -0.5f, -0.5f, //

        -0.5f, 0.5f,  -0.5f, 0.5f,  0.5f,  -0.5f, 0.5f,  0.5f,  0.5f,
        0.5f,  0.5f,  0.5f,  -0.5f, 0.5f,  0.5f,  -0.5f, 0.5f,  -0.5f, //
    };

    glGenVertexArrays(1, &vao_);
    glBindVertexArray(vao_);

    auto vbo = GLuint{};
    glGenBuffers(1, &vbo);
    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    glBufferData(GL_ARRAY_BUFFER, vertices.size() * sizeof(float), vertices.data(), GL_STATIC_DRAW);

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *)0);
    glEnableVertexAttribArray(0);

    glBindVertexArray(0);
    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0);
  }

  auto paintGL() -> void override {
    glClear(GL_COLOR_BUFFER_BIT);
    glBindVertexArray(vao_);

    auto model = QMatrix4x4{};
    model.rotate(angle_x_, {1, 0, 0});
    model.rotate(angle_y_, {0, 1, 0});
    auto view = QMatrix4x4{};
    view.translate(0, 0, -3);
    auto proj = QMatrix4x4{};
    proj.perspective(45, 1.0f * width() / height(), 0.1f, 100.0f);
    glUniformMatrix4fv(glGetUniformLocation(pro_, "model"), 1, GL_FALSE, model.data());
    glUniformMatrix4fv(glGetUniformLocation(pro_, "view"), 1, GL_FALSE, view.data());
    glUniformMatrix4fv(glGetUniformLocation(pro_, "proj"), 1, GL_FALSE, proj.data());

    glDrawArrays(GL_TRIANGLES, 0, 36);
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

protected:
  auto mousePressEvent(QMouseEvent *event) -> void override {
    if (event->button() == Qt::LeftButton) {
      last_pos_ = event->pos();
    }
  }

  auto mouseMoveEvent(QMouseEvent *event) -> void override {
    if (event->buttons() & Qt::LeftButton) {
      auto dx = event->position().x() - last_pos_.x();
      auto dy = event->position().y() - last_pos_.y();
      angle_x_ += dy;
      angle_y_ += dx;
      last_pos_ = event->pos();
      update();
    }
  }

private:
  GLuint vao_;
  GLuint pro_;
  QPoint last_pos_;
  float angle_x_;
  float angle_y_;

private:
  inline static auto v_shader_s = R"(#version 330 core
    layout (location=0) in vec3 pos;

    uniform mat4 model;
    uniform mat4 view;
    uniform mat4 proj;

    void main(){
      gl_Position = proj * view * model * vec4(pos, 1.0);
    })";

  inline static auto f_shader_s = R"(#version 330 core
    out vec4 color;

    void main(){
      color = vec4(1.0, 0.0, 0.0, 1.0);
    })";
};
```
