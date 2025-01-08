`ImGui::Begin(name, p_open, flag)` push 一个 imgui window。如果窗口处于 collapse（最小化），返回 false。

- name：窗口唯一标识，也是窗口标题，如果窗口不存在，则创建。

- p_open：`bool*`，如果使用，窗口右上角会出现关闭按钮，按钮点击后被设为 `false`（窗口不会自动关闭）。

`ImGui::End()` pop 一个 imgui window。必须和 `ImGui::Begin()` 保持相同数量。
