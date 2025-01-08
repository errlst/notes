# 漫反射贴图

光照场景中，纹理通常也被叫做漫反射贴图。

![](https://learnopengl-cn.github.io/img/02/04/container2.png)

# 镜面光贴图

镜面光通常是黑白色纹理，定义表面不同位置的镜面反射强度。

![](https://learnopengl-cn.github.io/img/02/04/container2_specular.png)

> 木头部分完全不会产生镜面反射。

# 代码

## 顶点着色器

```glsl
#version 330 core
layout(location = 0) in vec3 v_pos;
layout(location = 1) in vec3 v_normal;
layout(location = 2) in vec2 v_tex_coord;

out vec3 frag_pos;
out vec3 frag_normal;
out vec2 frag_tex_coord;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
uniform vec3 light_pos;

void main() {
    frag_pos = vec3(model * vec4(v_pos, 1.0));
    frag_normal = v_normal;
    frag_tex_coord = v_tex_coord;
    gl_Position = projection * view * model * vec4(v_pos, 1.0);
}
```

因为增加了纹理，因此需要纹理坐标作为额外输入。

## 片段着色器

```glsl
#version 330 core
out vec4 frag_color;

struct material_t {
    sampler2D diffuse;
    sampler2D specular;
    float shininess;
};

struct light_t {
    vec3 position;
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};

in vec3 frag_pos;
in vec3 frag_normal;
in vec2 frag_tex_coord;

uniform vec3 view_pos;
uniform material_t material;
uniform light_t light;

void main() {
    vec3 ambient = light.ambient * vec3(texture(material.diffuse, frag_tex_coord));

    vec3 norm = normalize(frag_normal);
    vec3 lightDir = normalize(light.position - frag_pos);
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, frag_tex_coord));

    vec3 viewDir = normalize(view_pos - frag_pos);
    vec3 reflectDir = reflect(-lightDir, norm);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    vec3 specular = light.specular * spec * vec3(texture(material.specular, frag_tex_coord));

    vec3 result = ambient + diffuse + specular;
    frag_color = vec4(result, 1.0);
}
```

环境光颜色通常和漫反射颜色一致，因此使用一个 `diffuse` 表示环境光和漫反射。

## cpp

```cpp
#include <GLFW/glfw3.h>
#include <glad/glad.h>

#include "camera.h"
#include "shader.h"

#include <imgui.h>
#include <imgui_impl_glfw.h>
#include <imgui_impl_opengl3.h>

shader_t cube_shader;
shader_t light_shader;

camera_t camera;
float camera_front;
float camera_right;

GLuint cube_vao;
GLuint light_vao;

bool mouse_down;

struct material_t {
    GLuint diffuse;
    GLuint specular;
    float shineness = 32;
} material;

auto camera_window() -> void {
    ImGui::Begin("camera");
    ImGui::Text("pos: (%.2f, %.2f, %.2f)", camera.position_.x, camera.position_.y, camera.position_.z);
    ImGui::Text("yaw: %.2f", camera.yaw_);
    ImGui::Text("pitch: %.2f", camera.pitch_);
    ImGui::Text("zoom");
    ImGui::SameLine();
    ImGui::SliderFloat("##slider", &camera.zoom_, 1, 45);
    ImGui::End();
}

auto material_window() -> void {
    ImGui::Begin("material");
    ImGui::Text("shineness");
    ImGui::SameLine();
    ImGui::SliderFloat("##slider", &material.shineness, 1, 256);
    ImGui::End();
}

auto gl_init() -> void {
    glEnable(GL_DEPTH_TEST);
    cube_shader.init("shader/cube.vs", "shader/cube.fs");
    light_shader.init("shader/light.vs", "shader/light.fs");

    float vertices[] = {
        -0.5f, -0.5f, -0.5f, 0.0f, 0.0f, -1.0f, 0.0f, 0.0f,
        0.5f, -0.5f, -0.5f, 0.0f, 0.0f, -1.0f, 1.0f, 0.0f,
        0.5f, 0.5f, -0.5f, 0.0f, 0.0f, -1.0f, 1.0f, 1.0f,
        0.5f, 0.5f, -0.5f, 0.0f, 0.0f, -1.0f, 1.0f, 1.0f,
        -0.5f, 0.5f, -0.5f, 0.0f, 0.0f, -1.0f, 0.0f, 1.0f,
        -0.5f, -0.5f, -0.5f, 0.0f, 0.0f, -1.0f, 0.0f, 0.0f,

        -0.5f, -0.5f, 0.5f, 0.0f, 0.0f, 1.0f, 0.0f, 0.0f,
        0.5f, -0.5f, 0.5f, 0.0f, 0.0f, 1.0f, 1.0f, 0.0f,
        0.5f, 0.5f, 0.5f, 0.0f, 0.0f, 1.0f, 1.0f, 1.0f,
        0.5f, 0.5f, 0.5f, 0.0f, 0.0f, 1.0f, 1.0f, 1.0f,
        -0.5f, 0.5f, 0.5f, 0.0f, 0.0f, 1.0f, 0.0f, 1.0f,
        -0.5f, -0.5f, 0.5f, 0.0f, 0.0f, 1.0f, 0.0f, 0.0f,

        -0.5f, 0.5f, 0.5f, -1.0f, 0.0f, 0.0f, 1.0f, 0.0f,
        -0.5f, 0.5f, -0.5f, -1.0f, 0.0f, 0.0f, 1.0f, 1.0f,
        -0.5f, -0.5f, -0.5f, -1.0f, 0.0f, 0.0f, 0.0f, 1.0f,
        -0.5f, -0.5f, -0.5f, -1.0f, 0.0f, 0.0f, 0.0f, 1.0f,
        -0.5f, -0.5f, 0.5f, -1.0f, 0.0f, 0.0f, 0.0f, 0.0f,
        -0.5f, 0.5f, 0.5f, -1.0f, 0.0f, 0.0f, 1.0f, 0.0f,

        0.5f, 0.5f, 0.5f, 1.0f, 0.0f, 0.0f, 1.0f, 0.0f,
        0.5f, 0.5f, -0.5f, 1.0f, 0.0f, 0.0f, 1.0f, 1.0f,
        0.5f, -0.5f, -0.5f, 1.0f, 0.0f, 0.0f, 0.0f, 1.0f,
        0.5f, -0.5f, -0.5f, 1.0f, 0.0f, 0.0f, 0.0f, 1.0f,
        0.5f, -0.5f, 0.5f, 1.0f, 0.0f, 0.0f, 0.0f, 0.0f,
        0.5f, 0.5f, 0.5f, 1.0f, 0.0f, 0.0f, 1.0f, 0.0f,

        -0.5f, -0.5f, -0.5f, 0.0f, -1.0f, 0.0f, 0.0f, 1.0f,
        0.5f, -0.5f, -0.5f, 0.0f, -1.0f, 0.0f, 1.0f, 1.0f,
        0.5f, -0.5f, 0.5f, 0.0f, -1.0f, 0.0f, 1.0f, 0.0f,
        0.5f, -0.5f, 0.5f, 0.0f, -1.0f, 0.0f, 1.0f, 0.0f,
        -0.5f, -0.5f, 0.5f, 0.0f, -1.0f, 0.0f, 0.0f, 0.0f,
        -0.5f, -0.5f, -0.5f, 0.0f, -1.0f, 0.0f, 0.0f, 1.0f,

        -0.5f, 0.5f, -0.5f, 0.0f, 1.0f, 0.0f, 0.0f, 1.0f,
        0.5f, 0.5f, -0.5f, 0.0f, 1.0f, 0.0f, 1.0f, 1.0f,
        0.5f, 0.5f, 0.5f, 0.0f, 1.0f, 0.0f, 1.0f, 0.0f,
        0.5f, 0.5f, 0.5f, 0.0f, 1.0f, 0.0f, 1.0f, 0.0f,
        -0.5f, 0.5f, 0.5f, 0.0f, 1.0f, 0.0f, 0.0f, 0.0f,
        -0.5f, 0.5f, -0.5f, 0.0f, 1.0f, 0.0f, 0.0f, 1.0f};

    GLuint vbo;
    glGenBuffers(1, &vbo);
    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    glGenVertexArrays(1, &cube_vao);
    glBindVertexArray(cube_vao);

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void *)0);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void *)(3 * sizeof(float)));
    glEnableVertexAttribArray(1);
    glVertexAttribPointer(2, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void *)(6 * sizeof(float)));
    glEnableVertexAttribArray(2);

    material.diffuse = shader_t::load_texture("./img/container.png");
    material.specular = shader_t::load_texture("./img/container_specular.png");
    cube_shader.use();
    cube_shader.set_uniform("material.diffuse", 0);
    cube_shader.set_uniform("material.specular", 1);

    glGenVertexArrays(1, &light_vao);
    glBindVertexArray(light_vao);

    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void *)0);
    glEnableVertexAttribArray(0);
}

auto gl_render() -> void {
    glClearColor(0.1f, 0.1f, 0.1f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    camera.move(camera_front, camera_right, ImGui::GetIO().DeltaTime);

    cube_shader.use();
    cube_shader.set_uniform("light.position", glm::vec3{1.2f, 1.0f, 2.0f});
    cube_shader.set_uniform("view_pos", camera.position_);

    auto light_color = glm::vec3{1, 1, 1};
    auto diffuse_color = light_color * glm::vec3(0.5f);
    auto ambient_color = diffuse_color * glm::vec3(0.2f);
    cube_shader.set_uniform("light.ambient", ambient_color);
    cube_shader.set_uniform("light.diffuse", diffuse_color);
    cube_shader.set_uniform("light.specular", glm::vec3{1.0f, 1.0f, 1.0f});

    cube_shader.set_uniform("material.shininess", material.shineness);

    auto projection = glm::perspective(glm::radians(camera.zoom_), (float)1920 / (float)1080, 0.1f, 100.0f);
    auto view = camera.view_matrix();
    cube_shader.set_uniform("projection", projection);
    cube_shader.set_uniform("view", view);

    auto model = glm::mat4(1.0f);
    cube_shader.set_uniform("model", model);

    glActiveTexture(GL_TEXTURE0);
    glBindTexture(GL_TEXTURE_2D, material.diffuse);
    glActiveTexture(GL_TEXTURE1);
    glBindTexture(GL_TEXTURE_2D, material.specular);

    glBindVertexArray(cube_vao);
    glDrawArrays(GL_TRIANGLES, 0, 36);

    light_shader.use();
    light_shader.set_uniform("projection", projection);
    light_shader.set_uniform("view", view);
    model = glm::mat4(1.0f);
    model = glm::translate(model, glm::vec3{1.2f, 1.0f, 2.0f});
    model = glm::scale(model, glm::vec3(0.2f));
    light_shader.set_uniform("model", model);

    glBindVertexArray(light_vao);
    glDrawArrays(GL_TRIANGLES, 0, 36);
}

auto key_callback(GLFWwindow *window, int key, int scancode, int action, int mods) -> void {
    switch (key) {
    case 'w':
    case 'W':
        camera_front = action ? 1 : 0;
        break;
    case 's':
    case 'S':
        camera_front = action ? -1 : 0;
        break;
    case 'a':
    case 'A':
        camera_right = action ? -1 : 0;
        break;
    case 'd':
    case 'D':
        camera_right = action ? 1 : 0;
        break;
    }
}

auto button_callback(GLFWwindow *window, int button, int action, int mods) -> void {
    mouse_down = (action == GLFW_PRESS && !ImGui::GetIO().WantCaptureMouse);
}

auto cursor_callback(GLFWwindow *window, double xpos, double ypos) -> void {
    static auto last_xpos = xpos;
    static auto last_ypos = ypos;
    if (mouse_down) {
        camera.rotate(xpos - last_xpos, last_ypos - ypos);
    }
    last_xpos = xpos;
    last_ypos = ypos;
}

int main() {
    glfwInit();
    auto window = glfwCreateWindow(1920, 1080, "opengl", NULL, NULL);
    glfwMakeContextCurrent(window);
    glfwSetKeyCallback(window, key_callback);
    glfwSetMouseButtonCallback(window, button_callback);
    glfwSetCursorPosCallback(window, cursor_callback);

    ImGui::CreateContext();
    ImGui::GetIO().Fonts->Clear();
    ImGui::GetIO().Fonts->AddFontFromFileTTF("./font/MapleMonoNL-Regular.ttf", 28.0f);
    ImGui_ImplOpenGL3_Init();
    ImGui_ImplGlfw_InitForOpenGL(window, true);

    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {
        return -1;
    }

    gl_init();

    while (!glfwWindowShouldClose(window)) {
        ImGui_ImplOpenGL3_NewFrame();
        ImGui_ImplGlfw_NewFrame();
        ImGui::NewFrame();

        gl_render();
        camera_window();
        material_window();

        ImGui::Render();
        ImGui_ImplOpenGL3_RenderDrawData(ImGui::GetDrawData());
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    return 0;
}
```
