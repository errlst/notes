## 立即渲染模式

早期 opengl 采用立即渲染模式，即固定管线模式，在 `glBegin` 和 `glEnd` 之间指定顶点和纹理坐标，一次发送一个坐标信息到 gpu，效率较低。

```cpp
class GLWidget : public QOpenGLWidget, protected QOpenGLExtraFunctions {
  public:
    GLWidget() {
        auto fmt = QSurfaceFormat{};
        fmt.setVersion(2, 2);
        setFormat(fmt);
    }

  public:
    auto initializeGL() -> void override {
        initializeOpenGLFunctions();
        glClearColor(0, 0, 0, 1);
    }

    auto paintGL() -> void override {
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
        // 设置投影范围
        glMatrixMode(GL_PROJECTION);
        glLoadIdentity();
        glOrtho(-10, 10, -10, 10, -1, 1);
        // 绘制
        glBegin(GL_LINE_LOOP);
        glColor3f(1, 1, 1);
        for (auto i = 0; i < 10; ++i) {
            glVertex2f(i, 0);
            glVertex2f(0, -i);
            glVertex2f(-i, 0);
            glVertex2f(0, i);
        }
        glEnd();
        glPopMatrix();
        glFlush();
    }

    auto resizeGL(int w, int h) -> void override { glViewport(0, 0, w, h); }
};

```

## 核心模式

opengl3.2 引入核心模式，通过 vao、vbo 等对象，一次将所有顶点发送到 gpu 中。

```cpp
class GLWidget : public QOpenGLWidget, protected QOpenGLExtraFunctions {
  public:
    GLWidget() {
        auto fmt = QSurfaceFormat{};
        fmt.setVersion(3, 3);
        setFormat(fmt);
    }

  public:
    auto initializeGL() -> void override {
        initializeOpenGLFunctions();
        glClearColor(255, 255, 255, 1);
        compileShader();

        auto vertics = std::vector<float>{};
        for (auto i = 0; i < 10; ++i) {
            vertics.emplace_back(i);
            vertics.emplace_back(0);
            vertics.emplace_back(1);

            vertics.emplace_back(0);
            vertics.emplace_back(-i);
            vertics.emplace_back(1);

            vertics.emplace_back(-i);
            vertics.emplace_back(0);
            vertics.emplace_back(1);

            vertics.emplace_back(0);
            vertics.emplace_back(i);
            vertics.emplace_back(1);
        }

        // 将顶点信息写入显存
        glGenBuffers(1, &vbo_);
        glGenVertexArrays(1, &vao_);
        glBindVertexArray(vao_);
        glBindBuffer(GL_ARRAY_BUFFER, vbo_);
        glBufferData(GL_ARRAY_BUFFER, vertics.size() * sizeof(float),
                     vertics.data(), GL_STATIC_DRAW);
        glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float),
                              nullptr);
        glEnableVertexAttribArray(0);
        glBindBuffer(GL_ARRAY_BUFFER, 0);
        glBindVertexArray(0);
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

    auto resizeGL(int w, int h) -> void override { glViewport(0, 0, w, h); }

  private:
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

        // 将 shader 链接为一个完整的管线
        program_ = glCreateProgram();
        glAttachShader(program_, v_shader);
        glAttachShader(program_, f_shader);
        glLinkProgram(program_);
        checkLink(program_);

        glDeleteShader(v_shader);
        glDeleteShader(f_shader);
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

    auto checkLink(GLuint pro) -> void {
        auto suc = 0;
        char log[1024];
        glGetProgramiv(pro, GL_LINK_STATUS, &suc);
        if (!suc) {
            glGetProgramInfoLog(pro, sizeof(log), nullptr, log);
            std::cout << log << "\n";
        }
    }

  private:
    GLuint vao_;
    GLuint vbo_;
    GLuint program_;

  private:
    // 顶点着色器 shader
    inline static auto v_shader_s =
        R"(#version 330 core
        layout (location = 0) in vec3 pos;
        uniform mat4 ortho;
        void main() {
            gl_Position = ortho * vec4(pos.x, pos.y, pos.z, 1.0);
        })";

    // 片段着色器 shader
    inline static auto f_shader_s =
        R"(#version 330 core
        out vec4 color;
        void main() {
            color = vec4(1.f, .5f, .2f, 1.f);
        })";
};
```

## 渲染管线

渲染管线的工作流程为：顶点数据(vertex data) -> 顶点着色器(vertex shader) -> 几何着色器(geometry shader) -> 图元装配(shape assembly) -> -> 光栅化(rasterization) -> 片段着色器(fragment shader) -> 测试混合。

其中顶点着色器、几何着色器和片段着色器可以通过 glsl(opengl shading language) 控制。使用核心模式时，至少需要定义顶点着色器和片段着色器。

#### 顶点着色器

顶点着色器的输入是所有顶点。

顶点着色器接受一个顶点数据，并对其进行一些基本处理。在着色器中通常进行坐标变换，使得坐标转换到标准设备坐标系中，标准设备坐标系的范围为 -1~1，任何落在范围外的坐标都会被抛弃。

```c
#version 330 core
layout (location = 0) in vec3 pos;
uniform mat4 ortho;
void main() {
    gl_Position = ortho * vec4(pos.x, pos.y, pos.z, 1.0);
}
```

- `layout (location = 0) in vec3 pos;`，定义输入变量。

- `uniform mat4 ortho;`，定义可以和 cpu 程序交互数据的变量。

- `gl_Position`，着色器输出变量。

#### 几何着色器

几何着色器接受一组顶点作为输入，并且能够通过产生新的顶点形成新的图元生成其他形状，并将新的图元输出到图元装配中。

#### 图元装配

图元装配将顶点着色器或几何着色器输出的顶点装配成指定的图元形状，如 `GL_LINE_LOOP` 为依次连接所有顶点。

#### 光栅化

opengl 中的所有事物都在 3D 空间中，光栅化负责将图元映射到屏幕上对应的像素，会拆切掉视图之外的所有像素，以提高执行效率。

#### 片段着色器

片段着色器的输入是所有像素，通过渲染管线及将顶点着色器的输出进行插值处理，并输入到片段着色器中，片段着色器根据输入计算出像素的最终颜色。

#### 混合测试

混合测试会测试片段对应的深度，判断该像素是否需要丢弃。同时也会检查 alpha 值，并进行 alpha 混合。
