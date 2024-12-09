# 反射

图形领域中通常不直接为物体定义颜色，而是定义物体从一个光源反射各种颜色分量的大小。

```cpp
auto light_color = QVector3{1.f, 1.f, 1.f};
auto obj_color = QVector{1.f, .5f, .3f};
auto res_color = light_color * obj_color;
```

为光源设置颜色，为物体设置吸收光的属性，模拟物理世界的效果。

