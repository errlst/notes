## how work

imgui 只负责输出顶点数据和命令列表，因此可以灵活的跨平台配合不同后端进行渲染。

### `ImDrawData`

imgui 暴露出的绘制结构为 `ImDrawData`：

```cpp
struct ImDrawData
{
    bool                    Valid;
    int                     CmdListsCount;
    int                     TotalIdxCount;
    int                     TotalVtxCount;
    ImVector<ImDrawList*>   CmdLists;
    ImVec2                  DisplayPos;
    ImVec2                  DisplaySize;
    ImVec2                  FramebufferScale;
    ImGuiViewport*          OwnerViewport;
};
```

- Valid：调用 `Render()` 和下一个 `NewFrame()` 之间有效。

- CmdListsCount：恒等于 CmdLists.size。

- TotalIdxCount：ImDrawList::IdxBuffer.size 的和。

- TotalVtxCount：ImDrawList::VtxBuffer.size 的和。

- CmdLists：所有需要渲染的 ImDrawList。

- DisplayPos：视口左上角位置。

- DisplaySize：视口大小。

- FramebufferScale：像素缩放。

### `ImDrawList`

每个 imgui 窗口维护一个自己的 `ImDrawList`，存储该窗口的绘制命令：

```cpp
struct ImDrawList
{
    // 渲染需要的数据
    ImVector<ImDrawCmd>     CmdBuffer;
    ImVector<ImDrawIdx>     IdxBuffer;
    ImVector<ImDrawVert>    VtxBuffer;
    ImDrawListFlags         Flags;
}
```

- CmdBuffer：一个 `ImDrawCmd` 对应一个 gpu 绘制调用，或者一个用户回调。

  > ```cpp
  > struct ImDrawCmd
  > {
  >     ImVec4          ClipRect;
  >     ImTextureID     TextureId;
  >     unsigned int    VtxOffset;
  >     unsigned int    IdxOffset;
  >     unsigned int    ElemCount;
  >
  >     ImDrawCallback  UserCallback;
  >     void*           UserCallbackData;
  >     int             UserCallbackDataSize;
  >     int             UserCallbackDataOffset;
  > };
  > ```
  >
  > - ClipRect：窗口尺寸，只有在坐标范围内的图元会被绘制。
  >
  > - TextureId：纹理 ID，由用户提供。
  >
  > - VtxOffset：VtxBuffer 中的起始偏移。
  >
  > - IdxOffset：IdxBuffer 中的起始偏移。如果渲染后端启用 `ImGuiBackendFlags_RendererHasVtxOffset` 标志，会使用索引数据。
  >
  > - ElemCount：索引数量，3 的倍数。
  >
  > - other：用户回调相关。

- IdxBuffer：索引数据。

- VtxBuffer：顶点数据。

  > ```cpp
  > struct ImDrawVert
  > {
  >     ImVec2  pos;
  >     ImVec2  uv;
  >     ImU32   col;
  > };
  > ```
  >
  > - uv：材质坐标。
  >
  > - col：rgba 颜色。

- Flags：抗锯齿设置。

## 示例

使用 glfw + opengl 后端：

```cpp
#include <GL/gl.h>
#include <GLFW/glfw3.h>
#include <imgui.h>
#include <imgui_impl_glfw.h>
#include <imgui_impl_opengl3.h>

auto main() -> int {
    // 初始化 glfw
    glfwInit();
    auto window = glfwCreateWindow(1920, 1080, "imgui", nullptr, nullptr);
    glfwMakeContextCurrent(window);

    // 初始化 imgui 和 backend
    ImGui::CreateContext();
    ImGui_ImplGlfw_InitForOpenGL(window, true);
    ImGui_ImplOpenGL3_Init();

    while (true) {
        // 事件驱动
        glfwPollEvents();
        if (glfwWindowShouldClose(window)) {
            break;
        }

        // 开始新的 frame
        ImGui_ImplOpenGL3_NewFrame();
        ImGui_ImplGlfw_NewFrame();
        ImGui::NewFrame();

        ImGui::ShowDemoWindow();

        // 结束 frame 并渲染
        ImGui::Render();
        ImGui_ImplOpenGL3_RenderDrawData(ImGui::GetDrawData());
        glfwSwapBuffers(window);
        glClear(GL_COLOR_BUFFER_BIT);
    }

    return 0;
}
```
