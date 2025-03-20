将光投射（cast）到物体的光源称为投光物（light caster）。

# 平行光

当光源处于无限远时，来自光源的所有光线就近似于平行。

平行光和点光源的主要区别是：平行光直接设置光源方向，点光源需要根据光源位置和表面位置计算光源方向。

```glsl
struct parallel_light_t {
    vec3 direction;
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};

vec3 calc_parallel_light(parallel_light_t light, vec3 norm, vec3 view_dir) {
    vec3 light_dir = normalize(-light.direction);
    vec3 ambient = light.ambient * vec3(texture(material.diffuse, frag_tex_coord));
    vec3 diffuse = light.diffuse * vec3(texture(material.diffuse, frag_tex_coord)) * max(dot(norm, light_dir), 0);
    vec3 specular = light.specular * vec3(texture(material.specular, frag_tex_coord)) * pow(max(dot(view_dir, reflect(-light_dir, norm)), 0), material.shininess);
    return (ambient + diffuse + specular);
}
```

# 点光源

点光源发出的光线通常是会随着距离逐渐衰减的，计算公式为：

$$
F_{att} = \frac{1}{K_c + K_l*d + K_q*d^2}
$$

- $K_c$、$K_l$、$K_q$ 是三个常数项，通常 $K_c$ 选为 1。

- $d$ 是光源和片段的距离。

```glsl
struct point_light_t {
    vec3 position;
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
    float k_constant;
    float k_linear;
    float k_quadratic;
};

vec3 calc_point_light(point_light_t light, vec3 norm, vec3 view_dir) {
    vec3 light_dir = normalize(light.position - frag_pos);
    vec3 ambient = light.ambient * vec3(texture(material.diffuse, frag_tex_coord));
    vec3 diffuse = light.diffuse * vec3(texture(material.diffuse, frag_tex_coord)) * max(dot(norm, light_dir), 0);
    vec3 specular = light.specular * vec3(texture(material.specular, frag_tex_coord)) * pow(max(dot(view_dir, reflect(-light_dir, norm)), 0), material.shininess);

    float d = length(light.position - frag_pos);
    float attenuation = 1.0 / (light.k_constant + light.k_linear * d + light.k_quadratic * pow(d, 2));
    return ambient + (diffuse + specular) * attenuation;
}
```

# 聚光

聚光是朝向特定方向的点光源，如手电筒。

聚光相对点光源通常需要额外两个属性：

- 聚光中心方向。

- 聚光半径的切光角（落在该角度内的片段才会被照亮）。

## 软化边缘

给聚光添加简单边缘平滑的方式可以使用内外两个聚光圆锥，当片段处于内外圆锥之间，给出 0~1 的光照强度。

```glsl
struct spot_light_t {
    vec3 position;
    vec3 direction;
    float inner_cut;
    float outer_cut;

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;

    float k_constant;
    float k_linear;
    float k_quadratic;
};

vec3 calc_spot_light(spot_light_t light, vec3 norm, vec3 view_dir) {
    vec3 light_dir = normalize(light.position - frag_pos);
    float theta = dot(light_dir, normalize(-light.direction));
    if (theta < light.outer_cut) {
        return vec3(0);
    }
    float intensity = clamp((theta - light.outer_cut) / (light.inner_cut - light.outer_cut), 0, 1);

    vec3 ambient = light.ambient * vec3(texture(material.diffuse, frag_tex_coord));
    vec3 diffuse = light.diffuse * vec3(texture(material.diffuse, frag_tex_coord)) * max(dot(norm, light_dir), 0);
    vec3 specular = light.specular * vec3(texture(material.specular, frag_tex_coord)) * pow(max(dot(view_dir, reflect(-light_dir, norm)), 0), material.shininess);

    float d = length(light.position - frag_pos);
    float attenuation = 1.0 / (light.k_constant + light.k_linear * d + light.k_quadratic * pow(d, 2));
    return (ambient + (diffuse + specular) * intensity) * attenuation;
}
```

> 角度计算为 cos 值，且在角度 0~180° 内，因此越小表示角度越大。
