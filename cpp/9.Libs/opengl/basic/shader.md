## 着色器对象

```cpp
auto compileShader() -> void {
    bool success;

    auto v_shader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(v_shader, 1, &v_shader_s, nullptr);
    glCompileShader(v_shader);
    checkCompile(v_shader);

    auto f_shader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(f_shader, 1, &f_shader_s, nullptr);
    glCompileShader(f_shader);
    checkCompile(f_shader);

    // ...
}

auto checkCompile(GLuint shader) -> void {
    auto suc = 0;
    char log[1024];
    glGetShaderiv(shader, GL_COMPILE_STATUS, &suc);
    if (!suc) {
        glGetShaderInfoLog(shader, sizeof(log), nullptr, log);
        std::cout << log << "\n";
    }
}
```

- `glCreateShader()`，创建着色器对象。

- `glSharedSource()`，将源码绑定到着色器对象。

- `glCompileShader()`，编译着色器对象。

  > `glCompileShader()` 不会返回编译信息，需要通过 `glGetShaderiv()` 获取着色器参数检查编译是否成功。
  >
  > 后缀 iv 表示返回参数是一个整数向量。

## 程序对象

程序对象是将多个着色器链接得到的最终版本。

```cpp
auto compileShader() -> void {
    // ...

    program_ = glCreateProgram();
    glAttachShader(program_, v_shader);
    glAttachShader(program_, f_shader);
    glLinkProgram(program_);
    checkLink(program_);

    glDeleteShader(v_shader);
    glDeleteShader(f_shader);
}

auto checkLink(GLuint pro) -> void {
    auto suc = 0;
    char log[1024];
    glGetProgramiv(pro, GL_LINK_STATUS, &suc);
    if (!suc) {
        glGetProgramInfoLog(pro, sizeof(log), nullptr, log);
        std::cout << log << "\n";
    }
}

auto paintGL() -> void override {
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    glOrtho(-10, 10, -10, 10, -1, 1);
    auto ortho = QMatrix4x4{};
    ortho.ortho(QRectF{QPointF{-10, 10}, QPointF{10, -10}});
    auto proj_loc = glGetUniformLocation(program_, "ortho");
    glUniformMatrix4fv(proj_loc, 1, GL_FALSE, ortho.data());
    glUseProgram(program_);
    glBindVertexArray(vao_);
    glDrawArrays(GL_LINE_LOOP, 0, 40);
}
```

- `glCreateProgram()`，创建程序对象。

- `glAttachShader()`，附加着色器。

- `glLinkProgram()`，链接着色器。

- `glDeleteShader()`，着色器对象链接到程序对象后，就不需要了，可以释放。

- `glUseProgram()`，激活程序对象，此后的渲染都会调用该程序对象。

## 顶点着色器

```c
#version 330 core
layout (location = 0) in vec3 pos;

void main() {
    gl_Position = vec4(pos.x, pos.y, pos.z, 1.0);
}
```

- 着色器的第一行必须是版本声明。

- `layout (localtion = 0)` 设定输入变量的位置值。

- `gl_Position` 是 glsl 预定义变量，其作为顶点着色器的输出。

# 简单 shader 类

```cpp
class Shader {
public:
  Shader(QOpenGLExtraFunctions *glfunc) : glfunc_(glfunc) {}

public:
  auto init(const char *v_shader_str, const char *f_shader_str) -> void {
    auto v_shader = glfunc_->glCreateShader(GL_VERTEX_SHADER);
    glfunc_->glShaderSource(v_shader, 1, &v_shader_str, nullptr);
    glfunc_->glCompileShader(v_shader);
    if (!check_compile(v_shader)) {
      return;
    }

    auto f_shader = glfunc_->glCreateShader(GL_FRAGMENT_SHADER);
    glfunc_->glShaderSource(f_shader, 1, &f_shader_str, nullptr);
    glfunc_->glCompileShader(f_shader);
    if (!check_compile(f_shader)) {
      return;
    }

    program_ = glfunc_->glCreateProgram();

    glfunc_->glAttachShader(program_, v_shader);
    glfunc_->glAttachShader(program_, f_shader);
    glfunc_->glLinkProgram(program_);
    check_link();

    glfunc_->glDeleteShader(v_shader);
    glfunc_->glDeleteShader(f_shader);
  }

  auto set_uniform(const std::string &name, const QMatrix4x4 &mat) -> void {
    glfunc_->glUniformMatrix4fv(get_loc(name), 1, GL_FALSE, mat.data());
  }

  auto set_uniform(const std::string &name, const QVector3D &vec) -> void {
    glfunc_->glUniform3f(get_loc(name), vec.x(), vec.y(), vec.z());
  }

  auto use() -> void { glfunc_->glUseProgram(program_); }

private:
  auto check_compile(GLuint shader) -> bool {
    auto suc = 0;
    char log[1024];
    glfunc_->glGetShaderiv(shader, GL_COMPILE_STATUS, &suc);
    if (!suc) {
      glfunc_->glGetShaderInfoLog(shader, sizeof(log), nullptr, log);
      std::println("{}", log);
    }
    return suc;
  }

  auto check_link() -> void {
    auto suc = 0;
    char log[1024];
    glfunc_->glGetProgramiv(program_, GL_LINK_STATUS, &suc);
    if (!suc) {
      glfunc_->glGetProgramInfoLog(program_, sizeof(log), nullptr, log);
      std::println("{}", log);
    }
  }

  auto get_loc(const std::string &name) -> GLuint {
    auto loc = glfunc_->glGetUniformLocation(program_, name.data());
    if (-1 == loc) {
      std::println("uniform {} not found", name);
    }
    return loc;
  }

private:
  GLuint program_;
  QOpenGLExtraFunctions *glfunc_;
};
```
