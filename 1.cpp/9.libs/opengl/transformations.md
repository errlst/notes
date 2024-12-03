## 齐次坐标

在标准三维坐标系中，位移操作不能直接通过 3x3 矩阵乘法实现。

使用齐次坐标，可以将缩放、位移、旋转等操作都通过 4x4 的矩阵运算实现，第四个分量称为 w，通常为 1。如果分量 w 为 0，则该坐标为一个方向向量，无法位移。

## 缩放

缩放向量 $(S_1,S_2,S_3)$ 对应的矩阵为：

$$
\left[
\begin{array}{}
S_1 & 0 & 0 & 0 \\
0 & S_2 & 0 & 0 \\
0 & 0 & S_3 & 0 \\
0 & 0 & 0 & 1
\end{array}
\right]

\cdot

\left(
\begin{array}{}
x \\
y \\
z \\
1
\end{array}
\right)

=

\left(
\begin{array}{}
S_1 \cdot x \\
S_2 \cdot y \\
S_3 \cdot z \\
1
\end{array}
\right)
$$

## 位移

位移向量 $(T_x,T_y,T_z)$ 对应的矩阵为：

$$
\left[
\begin{array}{}
1 & 0 & 0 & T_x \\
0 & 1 & 0 & T_y \\
0 & 0 & 0 & T_z \\
0 & 0 & 0 & 1
\end{array}
\right]
$$

## 旋转

在 3D 空间中旋转需要定义旋转角和旋转轴，旋转角可以是角度或者弧度。

绕旋转轴 $(R_x,R_y,R_z)$ 旋转 $\theta$ 角度，变换矩阵为：

$$
\left[
\begin{array}{}
\cos{\theta}+{R_x}^2(1-\cos{\theta}) & R_xR_y(1-\cos{\theta})-R_z\sin{\theta} & RxRz(1-\cos{\theta})+R_y\sin{\theta} & 0 \\

R_yR_x(1-\cos{\theta})+R_z\sin{\theta} & \cos{\theta}+{R_y}^2(1-\cos{\theta}) & R_yR_z(1-\cos{\theta})-R_x\sin{\theta} & 0 \\

R_zR_x(1-\cos{\theta})-R_y\sin{\theta} & R_zR_y(1-\cos{\theta})+R_x\sin{\theta} & \cos{\theta}+{R_z}^2(1-\cos{\theta}) & 0 \\

0 & 0 & 0 & 1
\end{array}
\right]
$$

## 组合

在进行变换组合时，需要将位移变换放在缩放变换之前，否则位移的向量也会同样被缩放。

## 示例

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
        compileShader();
        glClearColor(1, 1, 1, 1);

        auto vertics = std::vector<float>{
            .5,  -.5, 0, 1, 0, 0, // 右下
            -.5, -.5, 0, 0, 1, 0, // 左下
            0,   .5f, 0, 0, 0, 1, // 上
        };

        // 创建绑定 vao
        glGenVertexArrays(1, &vao_);
        glBindVertexArray(vao_);

        // 创建绑定 vbo
        auto vbo = GLuint{};
        glGenBuffers(1, &vbo);
        glBindBuffer(GL_ARRAY_BUFFER, vbo);
        glBufferData(GL_ARRAY_BUFFER, vertics.size() * sizeof(float), vertics.data(), GL_STATIC_DRAW);

        // 创建顶点属性指针
        glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *)0);
        glEnableVertexAttribArray(0);
        glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *)(3 * sizeof(float)));
        glEnableVertexAttribArray(1);
    }

    auto paintGL() -> void override {
        auto mat = QMatrix4x4{};
        mat.translate(x_, y_);
        glClear(GL_COLOR_BUFFER_BIT);
        glUseProgram(program_);
        glBindVertexArray(vao_);
        glUniformMatrix4fv(glGetUniformLocation(program_, "transform"), 1, GL_FALSE, mat.data());
        glDrawArrays(GL_TRIANGLES, 0, 3);

        x_ += x_v_;
        y_ += y_v_;
    }

    auto resizeGL(int w, int h) -> void override { glViewport(0, 0, w, h); }

  protected:
    auto keyPressEvent(QKeyEvent *e) -> void override {
        switch (e->key()) {
        case Qt::Key_Up:
            y_v_ += 0.0001;
            break;
        case Qt::Key_Down:
            y_v_ -= 0.0001;
            break;
        case Qt::Key_Left:
            x_v_ -= 0.0001;
            break;
        case Qt::Key_Right:
            x_v_ += 0.0001;
            break;
        }
    }

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
    GLuint program_;
    float x_ = 0;
    float y_ = 0;
    float x_v_ = 0;
    float y_v_ = 0;

  private:
    // 顶点着色器 shader
    inline static auto v_shader_s =
        R"(#version 330 core
        layout (location = 0) in vec3 pos;
        layout (location = 1) in vec3 v_color;

        out vec3 f_color;

        uniform mat4 transform;

        void main() {
            gl_Position = transform * vec4(pos, 1.0f);
            f_color = v_color;
        })";

    // 片段着色器 shader
    inline static auto f_shader_s =
        R"(#version 330 core
        in vec3 f_color;
        out vec4 frag_color;
        void main() {
            frag_color = vec4(f_color, 1.0f);
        })";
};

auto main(int argc, char *argv[]) -> int {
    QApplication app{argc, argv};
    GLWidget w;
    w.show();

    auto timer = new QTimer{};
    timer->callOnTimeout([&] { w.update(); });
    timer->start(0);

    return QApplication::exec();
}
```
