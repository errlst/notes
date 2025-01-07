典型的 glsl 结构如下：

```c
#version ver_num    // 版本声明

// 输入变量声明
in type in_var;

// 输出变量声明
out type out_var;

// uniform 变量声明
uniform type uni_var;

void main(){

}
```

对于顶点着色器来说，每个输入变量也被称为顶点属性，可以声明的顶点属性是有上限的，一般由硬件决定，opengl 确保至少可以声明 16 个包含 4 分量的顶点属性。可以通过查询 `GL_MAX_VERTEX_ATTRIBS` 查询具体上限。

## 输入输出

在两个着色器之间传递变量，变量名需要一致。即顶点着色器声明输出变量 `out vec3 v_color`，那么片段着色器对应的输入变量声明必须是 `in vec3 v_color`。

## uniform

uniform 变量是 cpu 传输数据到 gpu 上的一种方式，uniform 变量是全局的，可以在着色器程序的任意阶段访问，且必须唯一。

## layout

`layout (location = ?)` 是顶点着色器特有的对输入变量的声明，因为一个顶点包含的属性使用单个向量无法完全表示，通常需要声明多个输入变量。

如：在顶点属性中加上颜色值

```cpp
auto initializeGL() -> void override {
    // ...

    // 顶点属性：坐标 + 颜色
    auto vertics = std::vector<float>{
        .5,  -.5, 0, 1, 0, 0, // 右下
        -.5, -.5, 0, 0, 1, 0, // 左下
        0,   .5f, 0, 0, 0, 1, // 上
    };

    // ...

    // 顶点属性指针
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *)0);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *)(3 * sizeof(float)));
    glEnableVertexAttribArray(1);
}

auto paintGL() -> void override {
    glClear(GL_COLOR_BUFFER_BIT);
    glUseProgram(program_);
    glBindVertexArray(vao_);
    glDrawArrays(GL_TRIANGLES, 0, 3);
}

inline static auto v_shader_s =
    R"(#version 330 core
    layout (location = 0) in vec3 pos;
    layout (location = 1) in vec3 v_color;

    out vec3 f_color;

    void main() {
        gl_Position = vec4(pos, 1.0f);
        f_color = v_color;
    })";

inline static auto f_shader_s =
    R"(#version 330 core
    in vec3 f_color;
    out vec4 frag_color;
    void main() {
        frag_color = vec4(f_color, 1.0f);
    })";
```

## 向量

glsl 的向量是可以包含 2、3 或 4 个分量的容器，分量的类型是默认基础类型的任何一个：

| 类型    | 说明                     |
| ------- | ------------------------ |
| `vecn`  | 包含 n 分量的 float 向量 |
| `bvecn` | 包含 n 分量的 bool 向量  |
| `ivecn` | int 向量                 |
| `uvecn` | unsigned int 向量        |
| `dvecn` | double 向量              |

向量的分量可以通过 `.xyzw`、`.rgba` 和 `.stpq` 三种方式获取。

glsl 提供了重组的语法进行灵活的分量选择：

```c
vec2 v2;
vec4 v4 = v2.xyxy;
vec3 v3 = v4.xyz;
vec4 v4_2 = v4.xxxx + v4.yyyy;
vec4 v4_3 = vec4(v2, v3.x, v4.x);
```
