QML 使用类似 JS 表达式进行界面开发。

```qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    width: 400
    height: 400
    visible: true

    Button {
        id: button
        text: "a simple btn"
        background: Rectangle {
            implicitWidth: 100
            implicitHeight: 40
            color: button.down ? '#d6d6d6' : '#f6f6f6'
            border.color: '#26282a'
            border.width: 1
            radius: 4
        }
    }
}
```

通过 Cpp 加载 QML 代码，并渲染。

```cpp
#include <QGuiApplication>
#include <QQmlApplicationEngine>

auto main(int argc, char *argv[]) -> int {
  auto app = QGuiApplication{argc, argv};
  auto engine = QQmlApplicationEngine{};
  engine.load(QUrl::fromLocalFile("../main.qml"));
  return QGuiApplication::exec();
}
```

```cmake
cmake_minimum_required(VERSION 3.20)

project(quick_demo)

set(CMAKE_CXX_STANDARD 17)

add_executable(exec main.cc)

set(CMAKE_PREFIX_PATH "/home/jsr/Qt/6.8.1/gcc_64/lib/cmake")

find_package(Qt6 REQUIRED COMPONENTS Core Quick QuickControls2)

target_link_libraries(exec PRIVATE Qt6::Core Qt6::Quick Qt6::QuickControls2)

```

# 模块

使用 `import module_name version` 导入模块。从 Qt6.2 开始，导入模块时可以不指定版本，自动选择最新版本。

Qt 提供以下标准模块：

- QtQuick。QML 核心模块，提供基本的可视化项目、行为、动画等。

- QtQuick.Controls。基础 UI 控件。Qt6.0 之前分为 Controls 和 Controls2。Qt6.0 使用 Controls2 替代了 Controls，但加载库依然为 Controls2。

- QtQuick.Dialogs。标准对话框，如文件选择器，颜色选择器等。

- QtQuick.Layouts。布局管理器。

- QtQuick.Window。窗口管理功能。

- QtQuick.Particles。粒子效果。

- QtQuick.Shapes。基本图形绘制。

- QtGraphicalEffects。高级图形效果，如模糊、阴影等。

- QtMultimedia。多媒体处理。

- QtSensors。访问设备传感器，如加速器、陀螺仪等。

- QtLocation。地图和位置服务。

- QtWebEngine。渲染网页。
