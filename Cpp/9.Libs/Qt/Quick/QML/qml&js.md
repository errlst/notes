- [动态创建对象](#动态创建对象)
  - [createComponent](#createcomponent)
  - [createQmlObject](#createqmlobject)
  - [动态删除对象](#动态删除对象)
- [JS 资源](#js-资源)
  - [code-behind](#code-behind)
  - [shared library](#shared-library)
- [导入 QML](#导入-qml)

# 动态创建对象

有两种方式从 JS 动态创建 QML 对象：

- `Qt.createComponent()`，创建 `Component` 对象。用于创建已定义的现有组件。

- `Qt.createQmlObject()`，从 QML 字符串创建对象。用于创建运行时生成的 QML 对象。

## createComponent

`Qt.createComponent(url, mode, parent)`。

- url，QML 文档的 URL。

- mode，如果设置为 `Component.Asynchronous`，则异步加载。默认同步加载。

- parent，指定创建的 `Component` 对象父对象。

```js
// 点击添加方块
ApplicationWindow {
    visible: true
    width: 400
    height: 400

    Button {
        height: 50
        anchors.bottom: parent.bottom
        text: "load"
        autoRepeat: true
        onClicked: {
            if (!component) {
                // 异步加载
                component = Qt.createComponent("MyRectangle.qml", Component.Asynchronous);
                text = "loading";
                component.onStatusChanged.connect(function () {
                    // 加载完成
                    if (component.status == Component.Ready) {
                        text = "add";
                    }
                });
            }
            if (component.status == Component.Ready) {
                // 添加子对象
                component.createObject(container);
            }
        }

        property var component: undefined
    }

    Rectangle {
        width: parent.width
        height: parent.height - 50
        border.color: "gray"
        Flow {
            id: container
            width: parent.width
            height: parent.height
        }
    }
}
```

## createQmlObject

`Qt.createQmlObject(qml, parent, msg)`。

- qml，QML 字符串。

- parent，指定创建 QML 对象的父对象。如果在 QML 中导入了相对路径，则相对路径是相对 parent 的 QML 文档的路径。

- msg，编译报错时的额外提示。

如果创建成功，返回创建的对象，否则返回 `null`。

> 此函数会立即返回，因此如果 QML 字符串中加载了额外的外部组件，可能因为未加载完成而不起作用。

```js
// 左侧编辑 QML，右侧渲染结果
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    visible: true
    width: 400
    height: 400

    Rectangle {
        width: parent.width / 2
        height: parent.height
        border.color: "red"
        TextEdit {
            id: qml_input
            width: parent.width
            height: parent.height
        }
        Button {
            text: "draw"
            anchors.bottom: parent.bottom
            onClicked: {
                if (qml_result.current_obj) {
                    qml_result.current_obj.destroy();
                }
                qml_result.current_obj = Qt.createQmlObject(qml_input.text, qml_result);
            }
        }
    }

    Rectangle {
        id: qml_result
        property var current_obj: undefined
        width: parent.width / 2
        height: parent.height
        border.color: "red"
        anchors.right: parent.right
    }
}
```

## 动态删除对象

非 JS 动态创建的对象永远不应该手动删除，如果不需要，将其置为不可见即可。

可以在对象上调用 `.destroy(delay)` 销毁对象，delay 是可选的延迟销毁参数。

# JS 资源

JS 代码可以内联在 QML 中，也可以分离到单独的 JS 文件中，称为 JS 资源。QML 支持两种 JS 资源：

- code-behind implementation files，代码隐藏实现。

- shared library files。

## code-behind

默认 JS 文件视为 code-behind files，在这种情况下，文档中定义的 QML 对象类型每个实例都会维持一个单独的 JS 上下文状态。

<div style="display: flex; gap: 5px">

<div style="flex: 1">

```js
/// MyButton.js
var clickTimes = 0;
function onClicked(button) {
  ++clickTimes;
  console.log(clickTimes, button);
}
```

```js
/// MyButton.qml
import QtQuick
import QtQuick.Controls
import "MyButton.js" as MyButtonJS

Button {
    onClicked: MyButtonJS.onClicked(this)
}
```

每个 `MyButton` 单独维护自己的点击次数。

</div>

<div style="flex: 1">

```js
/// main.qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    visible: true
    width: 400
    height: 400

    Flow {
        MyButton {
            width: 50
            height: 50
            text: "btn1"
        }

        MyButton {
            width: 50
            height: 50
            text: "btn2"
        }
    }
}
```

</div>
</div>

## shared library

在 JS 文件中声明 `.pragma library`，此后该 JS 文件视为共享库，在所有 QML 文档中共享。

<div style="display: flex; gap: 5px;">

```js
/// MyButton.js
.pragma library // 该声明必须放在任何 JS 代码之前，除了注释

var clickTimes = 0;
function onClicked(button) {
  ++clickTimes;
  console.log(clickTimes, button);
}
```

所有 `MyButton` 共享点击次数。

</div>

# 导入 QML
