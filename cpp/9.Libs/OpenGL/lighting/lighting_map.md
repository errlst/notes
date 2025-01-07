# 漫反射贴图

漫反射贴图是表现了物体漫反射颜色的纹理图像。

## 着色器

<div style="display: flex; gap: 5px">

```glsl
/// cube.vert
#version 330 core
layout(location = 0) in vec3 pos;       // 顶点坐标
layout(location = 1) in vec3 normal;    // 顶点法向量
layout(location = 2) in vec2 tex_coord; // 贴图坐标

out vec3 frag_pos;
out vec3 frag_norm;
out vec2 frag_tex_coord;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main() {
  frag_pos = vec3(model * vec4(pos, 1.0)); // 转到世界坐标系
  frag_norm = normal;
  frag_tex_coord = tex_coord;
  gl_Position = projection * view * model * vec4(pos, 1.0);
}
```


```glsl
/// cube.frag
#version 330 core

struct Material {
  sampler2D diffuse;
  vec3 specular;
  float shininess;
};

in vec3 frag_pos;
in vec3 frag_norm;
in vec2 frag_tex_coord;

out vec4 color;

uniform vec3 light;     // 光源颜色
uniform vec3 light_pos; // 光源坐标
uniform vec3 view_pos;  // 相机坐标
uniform Material material;

void main() {
  // 环境光
  vec3 ambient = 0.1 * vec3(texture(material.diffuse, frag_tex_coord));

  // 漫反射
  vec3 norm = normalize(frag_norm);
  vec3 light_dir = normalize(light_pos - frag_pos);
  vec3 diff = light * vec3(texture(material.diffuse, frag_tex_coord)) * max(0, dot(norm, light_dir));

  // 镜面反射
  vec3 reflect_dir = reflect(-light_dir, norm);
  vec3 view_dir = normalize(view_pos - frag_pos);
  vec3 spec = light * material.specular * pow(max(dot(reflect_dir, view_dir), 0), material.shininess);

  color = vec4(ambient + diff + spec, 1);
}
```

</div>