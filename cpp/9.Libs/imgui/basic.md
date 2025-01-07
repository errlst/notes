## how work

imgui 只负责输出顶点数据和命令列表，因此可以灵活的跨平台配合不同后端进行渲染。

顶点结构为：

```cpp
struct ImDrawVert
{
    ImVec2  pos;
    ImVec2  uv;
    ImU32   col;
};
```
