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

#include <QApplication>
#include <QMatrix4x4>
#include <QOpenGLExtraFunctions>
#include <QOpenGLWidget>
#include <iostream>

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
        glClearColor(0, 0, 0, 1);
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

        glGenBuffers(1, &vbo_);
        glGenVertexArrays(1, &vao_);
        glBindVertexArray(vao_);
        glBindBuffer(GL_ARRAY_BUFFER_BINDING, vbo_);
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
    inline static auto v_shader_s =
        R"(#version 330 core
        layout (location = 0) in vec3 pos;
        uniform mat4 ortho;
        void main() {
            gl_Position = ortho * vec4(pos.x, pos.y, pos.z, 1.0);
        })";

    inline static auto f_shader_s =
        R"(#version 330 core
        out vec4 color;
        void main() {
            color = vec4(1, 1, 1, 1);
        })";
};

auto main(int argc, char *argv[]) -> int {
    QApplication app{argc, argv};
    GLWidget w;
    w.show();

    return QApplication::exec();
}
```