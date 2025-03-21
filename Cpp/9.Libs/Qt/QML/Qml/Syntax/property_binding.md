- [自动绑定](#自动绑定)
- [主动绑定](#主动绑定)

属性绑定允许指定不同对象不同属性之间的关系，当属性的依赖的值发生变化时，自动更新属性。

# 自动绑定

初始时为属性分配一个计算结果为所需值的 JS 表达式就会创建属性绑定。

```js
ApplicationWindow {
    visible: true
    width: 400
    height: 400

    Rectangle {
        // 绑定到 parent.width parent.height
        width: Math.min(parent.width, parent.height) / 2
        // 绑定到 width
        height: width
        anchors.centerIn: parent
        color: "gray"
    }
}
```

只要 JS 表达式中出现的属性值发生变化，就会触发更新属性。

# 主动绑定

如果为属性分配了静态值，将删除该绑定。

```js
ApplicationWindow {
    visible: true
    width: 400
    height: 400

    Rectangle {
        width: Math.min(parent.width, parent.height) / 2
        height: width
        anchors.centerIn: parent
        color: "gray"
        Keys.onSpacePressed: {
            width = parent.width;   // 属性绑定被删除
        }
        focus: true
    }
}
```

将表达式包装在 `Qt.binding()` 中，主动创建新的绑定关系。

```js
ApplicationWindow {
    visible: true
    width: 400
    height: 400

    Rectangle {
        width: Math.min(parent.width, parent.height) / 2
        height: width
        anchors.centerIn: parent
        color: "gray"
        Keys.onSpacePressed: {
            width = Qt.binding(function () {
                // 新的绑定关系
                return parent.width;
            });
        }
        focus: true
    }
}
```
