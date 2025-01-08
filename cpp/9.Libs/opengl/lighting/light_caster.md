将光投射（cast）到物体的光源称为投光物（light caster）。

# 平行光

当光源处于无限远时，来自光源的所有光线就近似于平行。

平行光和点光源的主要区别是：平行光直接设置光源方向，点光源需要根据光源位置和表面位置计算光源方向。

```glsl
#version 330 core
out vec4 frag_color;

struct material_t {
    sampler2D diffuse;
    sampler2D specular;
    float shininess;
};

struct light_t {
    // vec3 position;
    vec3 direction;
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
    // vec3 light_dir = normalize(light.position - frag_pos);
    vec3 light_dir = normalize(-light.direction);
    float diff = max(dot(norm, light_dir), 0.0);
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, frag_tex_coord));

    vec3 viewDir = normalize(view_pos - frag_pos);
    vec3 reflect_dir = reflect(-light_dir, norm);
    float spec = pow(max(dot(viewDir, reflect_dir), 0.0), material.shininess);
    vec3 specular = light.specular * spec * vec3(texture(material.specular, frag_tex_coord));

    vec3 result = ambient + diffuse + specular;
    frag_color = vec4(result, 1.0);
}
```

# 点光源

点光源发出的光线通常是会随着距离逐渐衰减的，计算公式为：

$$
F_{att} = \frac{1}{K_c + K_l*d + K_q*d^2}
$$

- $K_c$、$K_l$、$K_q$ 是三个常数项，是一些经验值+适当调整。

- $d$ 是点光源距离表面的距离。

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

    float k_constant;
    float k_linear;
    float k_quadratic;
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
    vec3 light_dir = normalize(light.position - frag_pos);
    float diff = max(dot(norm, light_dir), 0.0);
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, frag_tex_coord));

    vec3 viewDir = normalize(view_pos - frag_pos);
    vec3 reflect_dir = reflect(-light_dir, norm);
    float spec = pow(max(dot(viewDir, reflect_dir), 0.0), material.shininess);
    vec3 specular = light.specular * spec * vec3(texture(material.specular, frag_tex_coord));

    float distance = length(light.position - frag_pos);
    float attenuation = 1.0 / (light.k_constant + light.k_linear * distance + light.k_quadratic * (distance * distance));

    frag_color = vec4((ambient + diffuse + specular) * attenuation, 1.0);
}
```
