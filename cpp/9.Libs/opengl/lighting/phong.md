- [Phong Lighting Model](#phong-lighting-model)
  - [环境光照](#环境光照)
  - [漫反射光照](#漫反射光照)
  - [镜面反射](#镜面反射)
- [Gouraud Shading](#gouraud-shading)
- [示例](#示例)

# Phong Lighting Model

风氏光照模型由三个分量组成：

- 环境光照（Ambient Lighting）：即使在黑暗环境下，通常也存在一些自然光，因此使用一个环境光照常量，它永远会给物体一些颜色。

- 漫反射光照（Diffuse Lighting）：模拟光源对物体方向性的影响，即物体的某一部分越正对光源，其就越亮。

- 镜面光照（Specular Lighting）：模拟光泽物体上面出现的亮点。

  ![三种光照演示](https://learnopengl-cn.github.io/img/02/02/basic_lighting_phong.png)

因此风氏光照模型中，物体表面材质由以下四个属性描述：

```glsl
struct Material {
  vec3 ambient;   // 表面的环境光反射率
  vec3 diffuse;   // 表面的漫反射率
  vec3 specular;  // 表面的镜面反射率
  float shininess;// 表面的高光系数
};
```

表面颜色的计算公式为：

$$
I = I_a + I_d + I_s
$$

## 环境光照

环境光照的模拟很简单，只需要定义环境光照强度和环境光颜色即可。

环境光照的计算公式为：

> $$
> I_a = k_a \cdot L_a
> $$
>
> $k_a$：表面的环境光反射率。
>
> $L_a$：环境光颜色。

```glsl
vec3 amb = light * material.ambient;
```

## 漫反射光照

漫反射的计算基本单位为物体的面，计算漫反射光照需要：

- 法向量：垂直顶点表面的向量。

  > 顶点本身并没有表面，因此通常利用其周围的点来计算出该顶点的法向量。

- 定向的光线：光源位和片段位置向量差的方向向量。

![漫反射示意](https://learnopengl-cn.github.io/img/02/02/diffuse_light.png)

漫反射光的计算公式为：

$$
I_d = k_d \cdot L_d \cdot max(0, \vec{N} \cdot \vec{L})
$$

> $k_d$：表面的漫反射率。
>
> $L_d$：光源颜色。
>
> $\vec{R}$：表面法向量。
>
> $\vec{L}$：指向光源方向的单位向量。

```glsl
vec3 norm = normalize(f_normal);
vec3 light_dir = normalize(light_pos - f_pos);
vec3 diff = light * material.diffuse * max(0, dot(norm, light_dir));
```

## 镜面反射

镜面反射有三个计算因素：法向量、光方向和观察方向。

![镜面反射示意](https://learnopengl-cn.github.io/img/02/02/basic_lighting_specular_theory.png)

通过光方向和法向量计算光的出射方向，然后计算出射方向和观察方向的夹角，夹角越小则镜面反射因子越大。

> 因此通常也会以观察空间进行光照计算，可以零成本获取观察方向。

镜面反射光的计算公式为：

$$
I_s = k_s \cdot L_s \cdot max(0, \vec{R} \cdot \vec{V})^n
$$

> $k_s$：表面的镜面反射率。
>
> $L_s$：镜面光颜色。
>
> $\vec{R}$：反射向量，定义为 $\vec{R}=2(\vec{N} \cdot \vec{L}) - \vec{L}$。
>
> $\vec{V}$：指向观察者的方向向量。
>
> $n$：高光系数，值越大，光斑越集中。

```glsl
vec3 reflect_dir = reflect(-light_dir, norm);
vec3 view_dir = normalize(view_pos - f_pos);
vec3 spec = light * material.specular * pow(max(dot(reflect_dir, view_dir), 0), material.shininess);
```

# Gouraud Shading

在顶点着色器中实现风氏光照模型称为 Gouraud Shading，因为顶点数量远少于片段数量，因此这种方式效率更高。但是因为最后的颜色是通过插值得到的，光照效果更差。

![Gouraud & Phong](https://learnopengl-cn.github.io/img/02/02/basic_lighting_gouruad.png)

# 示例

<div style="display: flex; gap: 5px;">

```glsl
/// cube.vert
#version 330 core
layout(location = 0) in vec3 v_pos;    // 顶点坐标
layout(location = 1) in vec3 v_normal; // 顶点法向量

out vec3 f_pos;
out vec3 f_normal;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
uniform vec3 light_pos;

void main() {
  gl_Position = projection * view * model * vec4(v_pos, 1.0);
  f_pos = vec3(model * vec4(v_pos, 1.0)); // 转到世界坐标系
  f_normal = v_normal;
}
```

```glsl
/// cube.frag
#version 330 core

in vec3 f_pos;
in vec3 f_normal;

out vec4 color;

uniform vec3 light;     // 光源颜色
uniform vec3 light_pos; // 光源坐标
uniform vec3 view_pos;  // 相机坐标

struct Material {
  vec3 ambient;
  vec3 diffuse;
  vec3 specular;
  float shininess;
};
uniform Material material;

void main() {
  // 环境光
  vec3 amb = light * material.ambient;

  // 漫反射
  vec3 norm = normalize(f_normal);
  vec3 light_dir = normalize(light_pos - f_pos);
  vec3 diff = light * material.diffuse * max(0, dot(norm, light_dir));

  // 镜面反射
  vec3 reflect_dir = reflect(-light_dir, norm);
  vec3 view_dir = normalize(view_pos - f_pos);
  vec3 spec = light * material.specular * pow(max(dot(reflect_dir, view_dir), 0), material.shininess);

  color = vec4(amb + diff + spec, 1);
}
```

</div>

```glsl
/// light.frag
#version 330 core
out vec4 color;

uniform vec3 light;

void main() { color = vec4(light, 1); }
```

```cpp
/// main.cpp
auto light = QVector3D{1, 1, 1};
auto cube_color = QVector3D{.5, .5, .5};

auto ambient = QVector3D{.1, .1, .1};
auto diffuse = QVector3D{1, 1, 1};
auto specular = QVector3D{1, 1, 1};
auto shininess = 32;

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

    cube_shader_.init_from_file("./cube.vert", "./cube.frag");
    light_shader_.init_from_file("./cube.vert", "./light.frag");

    // 顶点和其法向量
    auto vertices = std::array{-0.5f, -0.5f, -0.5f, 0.0f,  0.0f,  -1.0f, 0.5f,  -0.5f, -0.5f, 0.0f,  0.0f,  -1.0f,
                               0.5f,  0.5f,  -0.5f, 0.0f,  0.0f,  -1.0f, 0.5f,  0.5f,  -0.5f, 0.0f,  0.0f,  -1.0f,
                               -0.5f, 0.5f,  -0.5f, 0.0f,  0.0f,  -1.0f, -0.5f, -0.5f, -0.5f, 0.0f,  0.0f,  -1.0f,

                               -0.5f, -0.5f, 0.5f,  0.0f,  0.0f,  1.0f,  0.5f,  -0.5f, 0.5f,  0.0f,  0.0f,  1.0f,
                               0.5f,  0.5f,  0.5f,  0.0f,  0.0f,  1.0f,  0.5f,  0.5f,  0.5f,  0.0f,  0.0f,  1.0f,
                               -0.5f, 0.5f,  0.5f,  0.0f,  0.0f,  1.0f,  -0.5f, -0.5f, 0.5f,  0.0f,  0.0f,  1.0f,

                               -0.5f, 0.5f,  0.5f,  -1.0f, 0.0f,  0.0f,  -0.5f, 0.5f,  -0.5f, -1.0f, 0.0f,  0.0f,
                               -0.5f, -0.5f, -0.5f, -1.0f, 0.0f,  0.0f,  -0.5f, -0.5f, -0.5f, -1.0f, 0.0f,  0.0f,
                               -0.5f, -0.5f, 0.5f,  -1.0f, 0.0f,  0.0f,  -0.5f, 0.5f,  0.5f,  -1.0f, 0.0f,  0.0f,

                               0.5f,  0.5f,  0.5f,  1.0f,  0.0f,  0.0f,  0.5f,  0.5f,  -0.5f, 1.0f,  0.0f,  0.0f,
                               0.5f,  -0.5f, -0.5f, 1.0f,  0.0f,  0.0f,  0.5f,  -0.5f, -0.5f, 1.0f,  0.0f,  0.0f,
                               0.5f,  -0.5f, 0.5f,  1.0f,  0.0f,  0.0f,  0.5f,  0.5f,  0.5f,  1.0f,  0.0f,  0.0f,

                               -0.5f, -0.5f, -0.5f, 0.0f,  -1.0f, 0.0f,  0.5f,  -0.5f, -0.5f, 0.0f,  -1.0f, 0.0f,
                               0.5f,  -0.5f, 0.5f,  0.0f,  -1.0f, 0.0f,  0.5f,  -0.5f, 0.5f,  0.0f,  -1.0f, 0.0f,
                               -0.5f, -0.5f, 0.5f,  0.0f,  -1.0f, 0.0f,  -0.5f, -0.5f, -0.5f, 0.0f,  -1.0f, 0.0f,

                               -0.5f, 0.5f,  -0.5f, 0.0f,  1.0f,  0.0f,  0.5f,  0.5f,  -0.5f, 0.0f,  1.0f,  0.0f,
                               0.5f,  0.5f,  0.5f,  0.0f,  1.0f,  0.0f,  0.5f,  0.5f,  0.5f,  0.0f,  1.0f,  0.0f,
                               -0.5f, 0.5f,  0.5f,  0.0f,  1.0f,  0.0f,  -0.5f, 0.5f,  -0.5f, 0.0f,  1.0f,  0.0f};

    // 方块
    glGenVertexArrays(1, &cube_vao_);
    glBindVertexArray(cube_vao_);

    auto vbo = GLuint{};
    glGenBuffers(1, &vbo);
    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    glBufferData(GL_ARRAY_BUFFER, vertices.size() * sizeof(float), vertices.data(), GL_STATIC_DRAW);

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *)0);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *)(3 * sizeof(float)));
    glEnableVertexAttribArray(1);

    // 光源
    glGenVertexArrays(1, &light_vao_);
    glBindVertexArray(light_vao_);

    glBindBuffer(GL_ARRAY_BUFFER, vbo); // 使用相同的 vbo

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *)0);
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
    cube_shader_.set_uniform("light", light);
    auto model = QMatrix4x4{};
    auto projection = QMatrix4x4{};
    projection.perspective(45.f, 1.0f * width() / height(), 0.1f, 100.0f);
    cube_shader_.set_uniform("model", model);
    cube_shader_.set_uniform("view", camera_.view_matrix());
    cube_shader_.set_uniform("projection", projection);
    cube_shader_.set_uniform("material.ambient", ambient);
    cube_shader_.set_uniform("material.diffuse", diffuse);
    cube_shader_.set_uniform("material.specular", specular);
    cube_shader_.set_uniform("material.shininess", shininess);
    cube_shader_.set_uniform("light_pos", QVector3D{1.2f, 1.0f, 2.0f});
    cube_shader_.set_uniform("view_pos", camera_.position_);
    glBindVertexArray(cube_vao_);
    glDrawArrays(GL_TRIANGLES, 0, 36);

    // 光源
    light_shader_.use();
    model.translate(1.2f, 1.0f, 2.0f);
    model.scale(0.2f);
    light_shader_.set_uniform("light", light);
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
```
