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

顶点着色器允许指定任何形式的顶点数据作为输入，因此必须在渲染前指定顶点数据的结构。

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
