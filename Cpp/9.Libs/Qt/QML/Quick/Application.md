`Application` 是单例类，将 `QApplication` 的部分属性提供给 QML 使用。

# 常用属性

- `arguments`，程序启动参数。

- `state`，程序状态。

- `name`，程序名。

- `version`，程序版本。

# 常用信号

- `aboutToQuit()`，程序即将退出事件循环时触发。

---

```qml
Window {
    visible: true
    width: 400
    height: 400

    Component.onCompleted: {
        console.log(Application.arguments);
    }

    title: `${Application.name} ${Application.version}`

    Connections {
        target: Application
        function onAboutToQuit() {
            console.log("app quit");
        }
    }
}
```
