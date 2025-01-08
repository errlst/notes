# 漫反射贴图

光照场景中，纹理通常也被叫做漫反射贴图。

## 着色器

```c
struct material_t {
    sampler2D diffuse;
    vec3 specular;
    float shininess;
};
```

> 环境光颜色通常都等于漫反射颜色，因此不分开存储。

<div style="display: flex; gap: 5px;" >

```c
/// cube.vs
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

```c
/// cube.fs
#version 330 core
out vec4 frag_color;

struct material_t {
    sampler2D diffuse;
    vec3 specular;
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
    vec3 specular = light.specular * (spec * material.specular);

    vec3 result = ambient + diffuse + specular;
    frag_color = vec4(result, 1.0);
}
```

</div>
