QtQuick 为用户界面提供了基本的构建块，如显示图像、文本、处理用户输入的对象。

以下是一个简单的颜色矩阵，但没有完整的窗口，只能通过 qml 预览：

```js
import QtQuick

Rectangle {
    width: 200
    height: 100
    color: 'red'

    Text {
        anchors.centerIn: parent
        text: "hello world!"
    }
}
```

# ApplicationWindow

`ApplicationWindow` 是 Controls 提供的程序起点。一个 `ApplicationWindow` 从上到下由 MenuBar、ToolBar、ContentArea、StatusBar 构成。

```js
import QtQuick.Controls

ApplicationWindow {
    visible: true

    title: "application window"
    width: 640
    height: 480

    menuBar: MenuBar {
        Menu {
            title: "memu 1"
            MenuItem {
                text: "menu item 1"
                onClicked: console.log("click menu item 1")
            }
            MenuItem {
                text: "menu itme 2"
                onClicked: console.log("click menu item 2")
            }
        }
        Menu {
            title: "memu 2"
        }
    }

    Button {
        text: "btn"
        onClicked: console.log("click btn")
        anchors.horizontalCenter: parent.horizontalCenter
        anchors.verticalCenter: parent.verticalCenter
    }
}
```