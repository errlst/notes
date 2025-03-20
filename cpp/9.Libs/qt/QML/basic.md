<link href="../../../../style.css", rel="stylesheet">

QML 使用 JS 表达式进行界面开发。通过 Cpp 加载 QML 代码，并渲染。

<div class="code_block">
<div>
qml/main.qml

```js
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    width: 400
    height: 400
    visible: true

    Button {
        id: button
        text: "a simple button"
        background: Rectangle {
            implicitWidth: 100
            implicitHeight: 50
            color: button.down ? '#0096fa' : '#96fafa'
            border.color: button.down ? '#96fafa' : '#0096fa'
            border.width: 1
            radius: 4
        }
    }
}

```

</div>
<div>
main.cpp

```cpp
#include <QGuiApplication>
#include <QQmlApplicationEngine>

auto main(int argc, char *argv[]) -> int {
  auto app = QGuiApplication{argc, argv};

  auto e = QQmlApplicationEngine{};
  e.load(QUrl::fromLocalFile("./qml/main.qml"));

  return QGuiApplication::exec();
}
```

</div>
<div>
xmake.lua

```lua
add_rules("mode.debug", "mode.release")

target("qml_demo")
    set_kind("binary")
    add_files("src/*.cpp")

    add_includedirs("/home/jsr/Qt/6.8.1/gcc_64/include")
    add_includedirs("/home/jsr/Qt/6.8.1/gcc_64/include/QtGui")
    add_includedirs("/home/jsr/Qt/6.8.1/gcc_64/include/QtQml")
    add_linkdirs("/home/jsr/Qt/6.8.1/gcc_64/lib")
    add_links("Qt6Core", "Qt6Gui", "Qt6Qml")

    after_build(function (target)
        local qml_src_dir = "src/qml"
        local qml_dst_dir = path.join(target:targetdir(), "qml")

        os.mkdir(qml_dst_dir)
        os.cp(path.join(qml_src_dir, "/*.qml"), qml_dst_dir)
    end)

    add_runenvs("LD_LIBRARY_PATH", "/home/jsr/Qt/6.8.1/gcc_64/lib")
```

</div>
</div>

## 模块

使用 `import module_name version` 导入模块。从 Qt6.2 开始，导入模块时可以不指定版本，自动选择最新版本。

Qt 提供以下标准模块：

- QtQuick。QML 核心模块，提供基本的可视化项目、行为、动画等。

- QtQuick.Controls。基础 UI 控件。Qt6.0 之前分为 Controls 和 Controls2。Qt6.0 使用 Controls2 替代了 Controls，但加载库依然为 Controls2。

- QtQuick.Dialogs。标准对话框，如文件选择器，颜色选择器等。

- QtQuick.Layouts。布局管理器。

- QtQuick.Window。窗口管理功能。

- QtQuick.Particles。粒子效果。

- QtQuick.Shapes。基本图形绘制。

- QtQuick.LocalStorage。SQLite 数据库接口。

- QtGraphicalEffects。高级图形效果，如模糊、阴影等。

- QtMultimedia。多媒体处理。

- QtSensors。访问设备传感器，如加速器、陀螺仪等。

- QtLocation。地图和位置服务。

## 工具

qmllint，验证 QML 文件语法的有效性。

qmlformat，QML 格式化工具。

qmlls，QML LSP。

qmlprofiler，QML 调试工具。

qml，快速加载 QML 文档。

qmlpreview，预览 QML，并实时观测 QML 文档的变化并重新加载。

> qmlpreview 并非直接加载 QML 文档，而是执行加载 QML 文档的程序，可以结合 qml 实现实时预览 QML 文档。
>
> ```shell
> qmlpreview qml target.qml
> ```
