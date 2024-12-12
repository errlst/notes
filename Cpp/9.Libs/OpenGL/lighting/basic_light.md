- [风氏光照模型](#风氏光照模型)
  - [环境光照](#环境光照)
  - [漫反射光照](#漫反射光照)
    - [计算法向量](#计算法向量)
    - [计算漫反射](#计算漫反射)
  - [镜面反射](#镜面反射)
- [Gouraud Shading](#gouraud-shading)

# 风氏光照模型

Phong Lighting Model 由三个分量组成：

- 环境光照（Ambient Lighting）：即使在黑暗环境下，通常也存在一些自然光，因此使用一个环境光照常量，它永远会给物体一些颜色。

- 漫反射光照（Diffuse Lighting）：模拟光源对物体方向性的影响，即物体的某一部分约正对光源，其就越亮。

- 镜面光照（Specular Lighting）：模拟光泽物体上面出现的亮点，镜面光照的颜色更倾向于光的颜色。

![三种光照演示](https://learnopengl-cn.github.io/img/02/02/basic_lighting_phong.png)

## 环境光照

环境光照的模拟很简单，只需要用光的颜色乘以常量环境因子，再乘以物体的颜色。

```glsl
/// 顶点着色器
#version 330 core
out vec4 color;

uniform vec3 cube_color;    // 物体颜色
uniform vec3 light_color;   // 光照颜色

void main() {
    float ambient_factor = 0.1;

    color = vec4(cube_color * ambient_factor, 1.0);
}
```

## 漫反射光照

漫反射的计算基本单位为物体的面，计算漫反射光照需要：

- 法向量：垂直顶点表面的向量。

- 定向的光线：光源位和片段位置向量差的方向向量。

![漫反射示意](https://learnopengl-cn.github.io/img/02/02/diffuse_light.png)

### 计算法向量

顶点本身并没有表面，因此通常利用其周围的点来计算出该顶点的法向量。

### 计算漫反射

<div style="display: flex; gap: 5px">

```glsl
/// 顶点着色器
#version 330 core
layout (location=0) in vec3 pos;
layout (location=1) in vec3 v_normal;   // 顶点法向量

out vec3 f_normal;
out vec3 f_pos;  // 顶点的世界位置

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
uniform vec3 light_pos; // 光源的世界位置

void main() {
    gl_Position = projection * view * model * vec4(pos, 1.0);
    f_normal = v_normal;
    f_pos = vec3(model * vec4(pos, 1.0));
}
```

```glsl
/// 片段着色器
#version 330 core
in vec3 f_normal;
in vec3 f_pos;

out vec4 color;

uniform vec3 cube_color;
uniform vec3 light_color;
uniform vec3 light_pos;

void main() {
    float ambient = 0.1;

    // 将标准化的法向量和光线方向进行点乘
    // 得到的值就作为漫反射分量
    // 且保证漫反射分量不能为0（即使光源没有照到物体表面，也不应该减少表面的颜色）
    vec3 norm = normalize(f_normal);
    vec3 light_dir = normalize(light_pos - f_pos);
    vec3 diffuse = max(dot(norm, light_dir), 0.0) * light_color;

    color = vec4(cube_color * (ambient + diffuse), 1.0);
}
```

</div>

## 镜面反射

镜面反射有三个计算因素：法向量、光方向和观察方向。

![镜面反射示意](https://learnopengl-cn.github.io/img/02/02/basic_lighting_specular_theory.png)

通过光方向和法向量计算光的出射方向，然后计算出射方向和观察方向的夹角，夹角越小则镜面反射因子越大。

> 因此通常也会以观察空间进行光照计算，可以零成本获取观察方向。

<div style="display: flex; gap: 5px;">

```glsl
/// 顶点着色器
#version 330 core
layout (location=0) in vec3 pos;
layout (location=1) in vec3 v_normal;

out vec3 f_normal;
out vec3 f_pos;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
uniform vec3 light_pos;

void main() {
    gl_Position = projection * view * model * vec4(pos, 1.0);
    f_normal = v_normal;
    f_pos = vec3(model * vec4(pos, 1.0));
};
```

```glsl
/// 片段着色器
#version 330 core
in vec3 f_normal;
in vec3 f_pos;

out vec4 color;

uniform vec3 cube_color;
uniform vec3 light_color;
uniform vec3 light_pos;
uniform vec3 view_pos;

void main() {
    float ambient = 0.1;

    vec3 norm = normalize(f_normal);
    vec3 light_dir = normalize(light_pos - f_pos);
    vec3 diffuse = max(dot(norm, light_dir), 0.0) * light_color;

    // 镜面强度
    float specular_strength = 0.5;
    // 看向的方向
    vec3 view_dir = normalize(view_pos - f_pos);
    // light_dir 是光的入射方向，因此要将其取反
    // 计算后的 reflect_dir 和 light_dir 相对法向量对称
    vec3 reflect_dir = reflect(-light_dir, norm);
    // 幂运算的指数为高光的反光度，反光度越大反色能力越强，高光点越小
    float spec = pow(max(dot(view_dir, reflect_dir), 0.0), 32);

    color = vec4(cube_color * (ambient + diffuse + specular_strength * spec * light_color), 1.0);
};
```

</div>

# Gouraud Shading

在顶点着色器中实现风氏光照模型称为 Gouraud Shading，因为顶点数量远少于片段数量，因此这种方式效率更高。但是因为最后的颜色是通过插值得到的，光照效果更差。

![Gouraud & Phong](https://learnopengl-cn.github.io/img/02/02/basic_lighting_gouruad.png)
