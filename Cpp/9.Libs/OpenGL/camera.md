一个摄像机由以下几个向量构成：世界空间中的坐标、观察的方向、指向相机右侧的向量和指向相机上方的向量。即一个以相机为原点的坐标系。

## 相机方向

按照惯例，相机右侧作为 +x 轴、相机上方作为 +y 轴，相机看向的方向作为 -z 轴。

获取指向摄像机 +z 轴的方向向量：

```cpp
auto camera_pos = QVector3D{1, 1, 1};
auto camera_target = QVector3D{0, 0, 0};
auto camera_direction = (camera_pos - camera_target).normalized();
```

## 上轴和右轴

通常直接定义 +y 轴，然后通过向量叉乘计算 +x 轴。

```cpp
auto camera_up = QVector3D{0, 1, 0};
auto camera_right = QVector3D::crossProduct(camera_up, camera_direct);
```

## LookAt

LookAt 矩阵可以将世界坐标转换到已经定义好的观察坐标中。LookAt 矩阵会创建一个看向目标的观察矩阵。

$$
LookAt

=

\left[
\begin{array}{}
R_x & R_y & R_z & 0 \\
U_x & U_y & U_z & 0 \\
D_x & D_y & D_z & 0 \\
0 & 0 & 0 & 1
\end{array}
\right]

*

\left[
\begin{array}{}
1 & 0 & 0 & -P_x \\
0 & 1 & 0 & -p_y \\
0 & 0 & 1 & -P_z \\
0 & 0 & 0 & 1
\end{array}
\right]
$$

其中 $R$、$U$、$D$ 分别是右向量、上向量和方向向量，$P$ 是相机位置向量。

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
        glUseProgram(program_);

        auto vertics = std::vector<float>{
            .5,  -.5, 0, 1, 0, 0, // 右下
            -.5, -.5, 0, 0, 1, 0, // 左下
            0,   .5f, 0, 0, 0, 1, // 上
        };

        glGenVertexArrays(1, &vao_);
        glBindVertexArray(vao_);

        auto vbo = GLuint{};
        glGenBuffers(1, &vbo);
        glBindBuffer(GL_ARRAY_BUFFER, vbo);
        glBufferData(GL_ARRAY_BUFFER, vertics.size() * sizeof(float), vertics.data(), GL_STATIC_DRAW);

        glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *)0);
        glEnableVertexAttribArray(0);
        glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *)(3 * sizeof(float)));
        glEnableVertexAttribArray(1);
    }

    auto paintGL() -> void override {
        auto model = QMatrix4x4{};
        model.translate(0, 0, 0);
        glUniformMatrix4fv(glGetUniformLocation(program_, "model"), 1, GL_FALSE, model.data());

        auto view = QMatrix4x4{};
        static float f = 0;
        f += 0.01;
        auto camera_x = (float)sin(f) * 5.f;
        auto camera_z = (float)cos(f) * 5.f;
        view.lookAt({camera_x, 0, camera_z}, {0, 0, 0}, {0, 1, 0});
        glUniformMatrix4fv(glGetUniformLocation(program_, "view"), 1, GL_FALSE, view.data());

        auto projection = QMatrix4x4{};
        projection.perspective(45.f, 1. * width() / height(), 0.1f, 100.f);
        glUniformMatrix4fv(glGetUniformLocation(program_, "projection"), 1, GL_FALSE, projection.data());

        glClear(GL_COLOR_BUFFER_BIT);
        glBindVertexArray(vao_);
        glDrawArrays(GL_TRIANGLES, 0, 3);
    }

    auto resizeGL(int w, int h) -> void override { glViewport(0, 0, w, h); }

  protected:
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
    inline static auto v_shader_s =
        R"(#version 330 core

        layout (location=0) in vec3 pos;
        layout (location=1) in vec3 color;

        out vec3 v_color;

        uniform mat4 model;
        uniform mat4 view;
        uniform mat4 projection;

        void main() {
            gl_Position = projection * view * model * vec4(pos, 1);
            v_color = color;
        })";

    inline static auto f_shader_s =
        R"(#version 330 core

        in vec3 v_color;
        out vec4 f_color;

        void main() {
            f_color = vec4(v_color, 1);
        })";
};
```
