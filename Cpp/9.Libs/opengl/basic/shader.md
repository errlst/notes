# 着色器对象

- `glCreateShader()`：创建 shader 对象。

- `glSharedSource()`：绑定 shader 代码。

- `glCompileShader()`：编译 shader 代码。

  > `glCompileShader()` 不会返回编译信息，需要通过 `glGetShaderiv()` 获取着色器参数检查编译是否成功。

- `glDeleteShader()`：释放 shader 对象，不影响链接后的程序对象。

# 程序对象

程序对象是将多个着色器链接得到的最终版本。

- `glCreateProgram()`：创建程序对象。

- `glAttachShader()`：添加着色器。

- `glLinkProgram()`，链接着色器。

- `glDeleteShader()`，着色器对象链接到程序对象后，就不需要了，可以释放。

- `glUseProgram()`，激活程序对象，此后的渲染都会调用该程序对象。

# 顶点着色器

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
#pragma once
#define STB_IMAGE_IMPLEMENTATION
#include <fstream>
#include <glad/glad.h>
#include <glm/glm.hpp>
#include <iostream>
#include <stb/stb_image.h>

class shader_t {
  public:
    auto init(std::string_view v_shader_path, std::string_view f_shader_path) -> void {
        auto v_shader = glCreateShader(GL_VERTEX_SHADER);
        auto v_shader_s = read_file(v_shader_path);
        auto v_shader_sp = v_shader_s.data();
        glShaderSource(v_shader, 1, &v_shader_sp, nullptr);
        glCompileShader(v_shader);
        check_compile(v_shader);

        auto f_shader = glad_glCreateShader(GL_FRAGMENT_SHADER);
        auto f_shader_s = read_file(f_shader_path);
        auto f_shader_sp = f_shader_s.data();
        glShaderSource(f_shader, 1, &f_shader_sp, nullptr);
        glCompileShader(f_shader);
        check_compile(f_shader);

        pro_ = glCreateProgram();
        glAttachShader(pro_, v_shader);
        glAttachShader(pro_, f_shader);
        glLinkProgram(pro_);
        check_link();

        glDeleteShader(v_shader);
        glDeleteShader(f_shader);
    }

    auto set_uniform(std::string_view name, glm::vec3 vec) -> void {
        glUniform3f(get_loc(name), vec.x, vec.y, vec.z);
    }

    auto set_uniform(std::string_view name, glm::mat4 mat) -> void {
        glUniformMatrix4fv(get_loc(name), 1, GL_FALSE, &mat[0][0]);
    }

    auto set_uniform(std::string_view name, float v) -> void {
        glUniform1f(get_loc(name), v);
    }

    auto set_uniform(std::string_view name, int v) -> void {
        glUniform1i(get_loc(name), v);
    }

    auto use() -> void { glUseProgram(pro_); }

    static auto load_texture(std::string_view path) -> GLuint {
        GLuint tex_id;
        glGenTextures(1, &tex_id);

        int width, height, nr;
        auto data = stbi_load(path.data(), &width, &height, &nr, 0);
        if (data) {
            GLenum fmt;
            if (1 == nr) {
                fmt = GL_RED;
            } else if (3 == nr) {
                fmt = GL_RGB;
            } else if (4 == nr) {
                fmt = GL_RGBA;
            }

            glBindTexture(GL_TEXTURE_2D, tex_id);
            glTexImage2D(GL_TEXTURE_2D, 0, fmt, width, height, 0, fmt, GL_UNSIGNED_BYTE, data);
            glGenerateMipmap(GL_TEXTURE_2D);

            glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
            glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
            glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
            glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_LINEAR);
            stbi_image_free(data);
        } else {
            std::cerr << "load texture failed " << path;
            exit(-1);
        }
        return tex_id;
    }

  private:
    auto read_file(std::string_view path) -> std::string {
        auto ifs = std::ifstream{path.data()};
        if (!ifs.is_open()) {
            std::cerr << "open failed " << path << "\n";
            exit(-1);
        }
        return {std::istreambuf_iterator<char>{ifs}, {}};
    }

    auto check_compile(GLuint shader) -> bool {
        auto suc = 0;
        char log[1024];
        glGetShaderiv(shader, GL_COMPILE_STATUS, &suc);
        if (!suc) {
            glGetShaderInfoLog(shader, sizeof(log), nullptr, log);
            std::cerr << "compile shader failed " << log << "\n";
            exit(-1);
        }
        return suc;
    }

    auto check_link() -> void {
        auto suc = 0;
        char log[1024];
        glGetProgramiv(pro_, GL_LINK_STATUS, &suc);
        if (!suc) {
            glGetProgramInfoLog(pro_, sizeof(log), nullptr, log);
            std::cerr << "link shader failed " << log << "\n";
            exit(-1);
        }
    }

    auto get_loc(std::string_view name) -> GLuint {
        auto loc = glGetUniformLocation(pro_, name.data());
        if (-1 == loc) {
            std::cerr << "not find uniform " << name << "\n";
            exit(-1);
        }
        return loc;
    }

  private:
    GLuint pro_;
};
```