## vbo

vertex buffer object，管理显存中存储的一组顶点数据。通过 vbo 可以批量将顶点数据传输到显卡中。

```cpp
auto vbo = 0;
glGenBuffers(1, &vbo);
glBindBuffer(GL_ARRAY_BUFFER, vbo);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertics), vertics, GL_STATIC_DRAW);
```

- `glGenBuffers()`，创建缓冲对象。

- `glBindBuffer()`，将缓冲对象绑定到具体的缓冲类型，`GL_ARRAY_BUFFER` 表示顶点缓冲对象。

- `glBufferData()`，将数据写入缓冲。第四个参数用于提示显卡如何优化缓冲区：

  - `GL_STATIC_DRAW`，数据几乎不会变化。

  - `GL_DYNAMIC_DRAW`，数据经常变化。

  - `GL_STREAM_DRAW`，数据每次绘制都会变化。

    > DRAW 表示这些数据用于绘制。

## 顶点属性

顶点着色器允许指定任何形式的顶点数据作为输入，因此必须在渲染前指定顶点数据的结构。坐标、颜色、贴图坐标等都属于顶点属性。

```cpp
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), nullptr);
glEnableVertexAttribArray(0);
```

- `glVertexAttribPointer(pos, size, type, normalized, strider, pointer)`，定义顶点属性指针。属性指针作用在调用该函数之前绑定的 vbo 对象上。

  - `pos`，顶点属性的位置值，对应 shader 中的 `layout (location = ?)`。

  - `size`，顶点的数据数量。

  - `type`，顶点的数据大小。

  - `normalized`，是否将所有数据标准化到 -1~1。

  - `strider`，每次解析一个顶点后移动的步长，如果数据是紧密排列，也可以传 0 让 opengl 决定步长。

  - `pointer`，起始位置偏移。

- `glEnableVertexAttribArray(pos)`，启用顶点属性指针。

## vao

vertex array object。vao 的作用是保存顶点属性指针，且顶点属性指针会指向 vbo。

## 绘制三角形

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

        auto vertics = std::vector<float>{
            -.5f, -.5f, 0, //
            .5f,  -.5f, 0, //
            0,    .5f,  0  //
        };

        // 创建绑定 vao
        glGenVertexArrays(1, &vao_);
        glBindVertexArray(vao_);

        // 创建绑定 vbo
        glGenBuffers(1, &vbo_);
        glBindBuffer(GL_ARRAY_BUFFER, vbo_);
        glBufferData(GL_ARRAY_BUFFER, vertics.size() * sizeof(float), vertics.data(), GL_STATIC_DRAW);

        // 创建顶点属性指针
        glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), nullptr);
        glEnableVertexAttribArray(0);

        // 解绑 vao 和 vbo
        // 顶点缓冲数据可以通过顶点属性指针访问，因此此时可以安全解绑 vbo
        glBindBuffer(GL_ARRAY_BUFFER, 0);
        glBindVertexArray(0);
    }

    auto paintGL() -> void override {
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
        glUseProgram(program_);
        glBindVertexArray(vao_);
        glDrawArrays(GL_TRIANGLES, 0, 40);
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
    // 顶点着色器 shader
    inline static auto v_shader_s =
        R"(#version 330 core
        layout (location = 0) in vec3 pos;
        void main() {
            gl_Position = vec4(pos.x, pos.y, pos.z, 1.0);
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

## 元素缓冲对象

在绘制模型时，存在大量重复的顶点，如果直接使用顶点数据进行绘制，会产生大量额外开销。element buffer object 存储顶点的索引，此时重复的顶点只需要一份数据即可。

和 vbo 不同，vao 会直接存储 ebo 的引用，因此如果在 vao 解绑之前解绑 ebo，那么这个 ebo 的配置就会消失。

使用 ebo 进行绘制时，调用 `glDrawElements()`。

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

        auto vertics = std::vector<float>{
            -.5f, .5f,  0, // 左上角
            .5f,  .5f,  0, // 右上角
            -.5f, -.5f, 0, // 左下角
            .5f,  -.5f, 0, // 右下角
        };
        auto indices = std::vector<int>{
            0, 1, 2, // 上半三角
            1, 2, 3, // 下半三角
        };

        // 创建绑定 vao
        glGenVertexArrays(1, &vao_);
        glBindVertexArray(vao_);

        // 创建绑定 vbo
        auto vbo = GLuint{};
        glGenBuffers(1, &vbo);
        glBindBuffer(GL_ARRAY_BUFFER, vbo);
        glBufferData(GL_ARRAY_BUFFER, vertics.size() * sizeof(float), vertics.data(), GL_STATIC_DRAW);

        // 创建绑定 ebo
        auto ebo = GLuint{};
        glGenBuffers(1, &ebo);
        glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo);
        glBufferData(GL_ELEMENT_ARRAY_BUFFER, indices.size() * sizeof(int), indices.data(), GL_STATIC_DRAW);

        // 创建顶点属性指针
        glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), nullptr);
        glEnableVertexAttribArray(0);

        // vao 会存放 ebo 的绑定，因此解绑 ebo 需要放在解绑 vao 之后
        glBindVertexArray(0);
        glBindBuffer(GL_ARRAY_BUFFER, 0);
        glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0);

        // 使用线框绘制
        glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);
    }

    auto paintGL() -> void override {
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
        glUseProgram(program_);
        glBindVertexArray(vao_);
        glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, nullptr);
        glBindVertexArray(0);
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
    GLuint program_;

  private:
    // 顶点着色器 shader
    inline static auto v_shader_s =
        R"(#version 330 core
        layout (location = 0) in vec3 pos;
        void main() {
            gl_Position = vec4(pos.x, pos.y, pos.z, 1.0);
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
