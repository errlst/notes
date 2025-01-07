# 反射

图形领域中通常不直接为物体定义颜色，而是定义物体从一个光源反射各种颜色分量的大小。

```cpp
auto light_color = QVector3{1.f, 1.f, 1.f};
auto obj_color = QVector{1.f, .5f, .3f};
auto res_color = light_color * obj_color;
```

为光源设置颜色，为物体设置吸收光的属性，模拟物理世界的效果。

# 示例

```cpp
// 显示一个普通立方体和光源立方体
// 但没有实现光照算法

auto cube_v_shader = R"(#version 330 core
layout (location=0) in vec3 pos;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main() {
    gl_Position = projection * view * model * vec4(pos, 1.0);
})";

auto cube_f_shader = R"(#version 330 core
out vec4 color;

uniform vec3 cube_color;
uniform vec3 light_color;

void main() {
    color = vec4(light_color * cube_color, 1.0);
})";

auto light_f_shader = R"(#version 330 core
out vec4 color;

void main() {
    color = vec4(1.0);  // 白色光源
})";

class GLWindow : public QOpenGLWindow, protected QOpenGLExtraFunctions {
public:
  GLWindow() : cube_shader_(this), light_shader_(this) {
    setSurfaceType(QSurface::OpenGLSurface);
    QSurfaceFormat format;
    format.setProfile(QSurfaceFormat::CoreProfile);
    format.setVersion(3, 3);
    format.setDepthBufferSize(24);
    setFormat(format);
  }

  auto initializeGL() -> void override {
    initializeOpenGLFunctions();
    glEnable(GL_DEPTH_TEST);

    cube_shader_.init(cube_v_shader, cube_f_shader);
    light_shader_.init(cube_v_shader, light_f_shader);

    auto vertices = std::array{
        -0.5f, -0.5f, -0.5f, 0.5f,  -0.5f, -0.5f, 0.5f,  0.5f,  -0.5f,
        0.5f,  0.5f,  -0.5f, -0.5f, 0.5f,  -0.5f, -0.5f, -0.5f, -0.5f,

        -0.5f, -0.5f, 0.5f,  0.5f,  -0.5f, 0.5f,  0.5f,  0.5f,  0.5f,
        0.5f,  0.5f,  0.5f,  -0.5f, 0.5f,  0.5f,  -0.5f, -0.5f, 0.5f,

        -0.5f, 0.5f,  0.5f,  -0.5f, 0.5f,  -0.5f, -0.5f, -0.5f, -0.5f,
        -0.5f, -0.5f, -0.5f, -0.5f, -0.5f, 0.5f,  -0.5f, 0.5f,  0.5f,

        0.5f,  0.5f,  0.5f,  0.5f,  0.5f,  -0.5f, 0.5f,  -0.5f, -0.5f,
        0.5f,  -0.5f, -0.5f, 0.5f,  -0.5f, 0.5f,  0.5f,  0.5f,  0.5f,

        -0.5f, -0.5f, -0.5f, 0.5f,  -0.5f, -0.5f, 0.5f,  -0.5f, 0.5f,
        0.5f,  -0.5f, 0.5f,  -0.5f, -0.5f, 0.5f,  -0.5f, -0.5f, -0.5f,

        -0.5f, 0.5f,  -0.5f, 0.5f,  0.5f,  -0.5f, 0.5f,  0.5f,  0.5f,
        0.5f,  0.5f,  0.5f,  -0.5f, 0.5f,  0.5f,  -0.5f, 0.5f,  -0.5f,
    };

    // 方块
    glGenVertexArrays(1, &cube_vao_);
    glBindVertexArray(cube_vao_);

    auto vbo = GLuint{};
    glGenBuffers(1, &vbo);
    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    glBufferData(GL_ARRAY_BUFFER, vertices.size() * sizeof(float), vertices.data(), GL_STATIC_DRAW);

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *)0);
    glEnableVertexAttribArray(0);

    // 光源
    glGenVertexArrays(1, &light_vao_);
    glBindVertexArray(light_vao_);

    glBindBuffer(GL_ARRAY_BUFFER, vbo); // 使用相同的 vbo

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *)0);
    glEnableVertexAttribArray(0);

    auto timer = new QTimer{};
    timer->callOnTimeout([this] { this->update(); });
    timer->start(33);
  }

  auto resizeGL(int w, int h) -> void override { glViewport(0, 0, w, h); }

  auto paintGL() -> void override {
    glClearColor(0, 0, 0, 1);
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

    camera_.move(direction_, 0.033);
    camera_.rotate(x_offset_, y_offset_);
    x_offset_ = y_offset_ = 0;

    // 方块
    cube_shader_.use();
    cube_shader_.set_uniform("cube_color", QVector3D(1.0f, 0.5f, 0.31f));
    cube_shader_.set_uniform("light_color", QVector3D(1.0f, 1.0f, 1.0f));
    auto model = QMatrix4x4{};
    auto projection = QMatrix4x4{};
    projection.perspective(45.f, 1.0f * width() / height(), 0.1f, 100.0f);
    cube_shader_.set_uniform("model", model);
    cube_shader_.set_uniform("view", camera_.view_matrix());
    cube_shader_.set_uniform("projection", projection);
    glBindVertexArray(cube_vao_);
    glDrawArrays(GL_TRIANGLES, 0, 36);

    // 光源
    light_shader_.use();
    model.translate(1.2f, 1.0f, 2.0f);
    model.scale(0.2f);
    light_shader_.set_uniform("model", model);
    light_shader_.set_uniform("view", camera_.view_matrix());
    light_shader_.set_uniform("projection", projection);
    glBindVertexArray(light_vao_);
    glDrawArrays(GL_TRIANGLES, 0, 36);
  }

protected:
  auto keyPressEvent(QKeyEvent *event) -> void override {
    switch (event->key()) {
    case Qt::Key_W:
      direction_.front += 1;
      break;
    case Qt::Key_S:
      direction_.front -= 1;
      break;
    case Qt::Key_A:
      direction_.right -= 1;
      break;
    case Qt::Key_D:
      direction_.right += 1;
      break;
    default:
      break;
    }
  }

  auto keyReleaseEvent(QKeyEvent *event) -> void override {
    switch (event->key()) {
    case Qt::Key_W:
      direction_.front -= 1;
      break;
    case Qt::Key_S:
      direction_.front += 1;
      break;
    case Qt::Key_A:
      direction_.right += 1;
      break;
    case Qt::Key_D:
      direction_.right -= 1;
      break;
    default:
      break;
    }
  }
  auto mousePressEvent(QMouseEvent *event) -> void override {
    if (event->button() == Qt::LeftButton) {
      last_pos_ = event->pos();
    }
  }

  auto mouseMoveEvent(QMouseEvent *event) -> void override {
    if (event->buttons() & Qt::LeftButton) {
      x_offset_ = event->position().x() - last_pos_.x();
      y_offset_ = last_pos_.y() - event->position().y();
      last_pos_ = event->pos();
    }
  }

private:
  Shader cube_shader_;
  GLuint cube_vao_;

  Shader light_shader_;
  GLuint light_vao_;

  Camera camera_;
  CameraDirection direction_{0, 0};
  QPoint last_pos_;
  float x_offset_ = 0;
  float y_offset_ = 0;
};

auto main(int argc, char *argv[]) -> int {
  auto app = QGuiApplication{argc, argv};
  auto w = GLWindow{};
  w.show();
  return QGuiApplication::exec();
}
```
