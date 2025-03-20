```cpp
struct ImGuiIO {
    ImGuiConfigFlags  ConfigFlags;
    ImGuiBackendFlags BackendFlags;
    ImVec2            DisplaySize;   // 一般等于视口大小
    float             DeltaTime;     // 上一个渲染帧的间隔
    float             IniSavingRate; // 保存 .ini 文件的最小时间
    const char       *IniFilename;   // .ini 文件路径
    const char       *LogFilename;   // .log 文件路径
    void             *UserData;

    // 字体系统
    ImFontAtlas *Fonts;                   // 所有字体
    float        FontGlobalScale;         // 全局字体缩放
    bool         FontAllowUserScaling;    // 通过 ctrl+whell 缩放单个窗口字体（此缩放和全局字体缩放独立计算）
    ImFont      *FontDefault;             // 默认使用字体
    ImVec2       DisplayFramebufferScale; // 全局显示缩放，最后输出到 ImDrawData::FrameBufferScale

    // 键盘/手柄设置
    bool ConfigNavSwapGamepadButtons;
    bool ConfigNavMoveSetMousePos;
    bool ConfigNavCaptureKeyboard;
    bool ConfigNavEscapeClearFocusItem;
    bool ConfigNavEscapeClearFocusWindow;
    bool ConfigNavCursorVisibleAuto;
    bool ConfigNavCursorVisibleAlways;

    // 杂项
    bool  MouseDrawCursor;
    bool  ConfigMacOSXBehaviors;
    bool  ConfigInputTrickleEventQueue;
    bool  ConfigInputTextCursorBlink;
    bool  ConfigInputTextEnterKeepActive;
    bool  ConfigDragClickToInputText;
    bool  ConfigWindowsResizeFromEdges;
    bool  ConfigWindowsMoveFromTitleBarOnly;
    bool  ConfigWindowsCopyContentsWithCtrlC;
    bool  ConfigScrollbarScrollByPage;
    float ConfigMemoryCompactTimer;

    // 输入行为
    float MouseDoubleClickTime;
    float MouseDoubleClickMaxDist;
    float MouseDragThreshold;
    float KeyRepeatDelay;
    float KeyRepeatRate;

    // backend 数据
    const char *BackendPlatformName;
    const char *BackendRendererName;
    void       *BackendPlatformUserData;
    void       *BackendRendererUserData;
    void       *BackendLanguageUserData;

    // 输入函数，NewFrame() 前调用
    IMGUI_API void AddKeyEvent(ImGuiKey key, bool down);
    IMGUI_API void AddKeyAnalogEvent(ImGuiKey key, bool down, float v);
    IMGUI_API void AddMousePosEvent(float x, float y);
    IMGUI_API void AddMouseButtonEvent(int button, bool down);
    IMGUI_API void AddMouseWheelEvent(float wheel_x, float wheel_y);
    IMGUI_API void AddMouseSourceEvent(ImGuiMouseSource source);
    IMGUI_API void AddFocusEvent(bool focused);
    IMGUI_API void AddInputCharacter(unsigned int c);
    IMGUI_API void AddInputCharacterUTF16(ImWchar16 c);
    IMGUI_API void AddInputCharactersUTF8(const char *str);
    IMGUI_API void SetKeyEventNativeData(ImGuiKey key, int native_keycode, int native_scancode, int native_legacy_index = -1);
    IMGUI_API void SetAppAcceptingEvents(bool accepting_events);
    IMGUI_API void ClearEventsQueue();
    IMGUI_API void ClearInputKeys();
    IMGUI_API void ClearInputMouse();

    // 输出变量，NewFrame()、Render() 更新
    bool   WantCaptureMouse;      // 是否需要鼠标输入
    bool   WantCaptureKeyboard;   // 是否需要键盘输入
    bool   WantTextInput;         // 提供给移动平台，此时需要显示一个软键盘
    bool   WantSetMousePos;       // 下一帧需要重新获取鼠标位置
    bool   WantSaveIniSettings;   // 提示需要保存设置（如果设置了手动保存设置）
    bool   NavActive;             // 键盘/手柄导航激活，即窗口聚焦，且没有设置 ImGuiWindowFlags_NoNavInputs
    bool   NavVisible;            // 键盘/手柄导航高亮可见且允许
    float  Framerate;             // 基于 DeltaTime 计算的帧率
    int    MetricsRenderVertices; // 上次渲染输出的顶点数量
    int    MetricsRenderIndices;  // 上次渲染输出的索引数量
    int    MetricsRenderWindows;  // 可见窗口数量
    int    MetricsActiveWindows;  // 活动窗口数量
    ImVec2 MouseDelta;

    // 内部变量，不保证兼容性
    ImGuiContext     *Ctx; // 所属上下文
    ImVec2            MousePos;
    bool              MouseDown[5];
    float             MouseWheel;
    float             MouseWheelH;
    ImGuiMouseSource  MouseSource;
    bool              KeyCtrl;
    bool              KeyShift;
    bool              KeyAlt;
    bool              KeySuper;
    ImGuiKeyChord     KeyMods;
    ImGuiKeyData      KeysData[ImGuiKey_NamedKey_COUNT];
    bool              WantCaptureMouseUnlessPopupClose;
    ImVec2            MousePosPrev;
    ImVec2            MouseClickedPos[5];
    double            MouseClickedTime[5];
    bool              MouseClicked[5];
    bool              MouseDoubleClicked[5];
    ImU16             MouseClickedCount[5];
    ImU16             MouseClickedLastCount[5];
    bool              MouseReleased[5];
    bool              MouseDownOwned[5];
    bool              MouseDownOwnedUnlessPopupClose[5];
    bool              MouseWheelRequestAxisSwap;
    bool              MouseCtrlLeftAsRightClick;
    float             MouseDownDuration[5];
    float             MouseDownDurationPrev[5];
    float             MouseDragMaxDistanceSqr[5];
    float             PenPressure;
    bool              AppFocusLost;
    bool              AppAcceptingEvents;
    ImWchar16         InputQueueSurrogate;
    ImVector<ImWchar> InputQueueCharacters;
};
```