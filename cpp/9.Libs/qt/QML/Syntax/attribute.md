<link href="../../../../../style.css", rel="stylesheet">

## id

id 由 QML 语言自身提供，每个 QML 对象只能有一个 id，且在组件范围内唯一。创建对象实例后就无法修改 id，且 id 无法适用于某些语法，如 `.id` 是非法的。

id 必须以小写字母或下划线开头，且只能由字母、数字或下划线组成。

## property

可以为 property 分配静态值或绑定到表达式。QML 的属性绑定和 JS 的赋值是不同的，绑定是一个协议，且在整个生命周期有效，而 JS 的赋值只会产生一次效果。
当一个新的绑定生效或者给属性赋值时，绑定的声明周期就会结束。

在 C++中，可以通过 `Q_PROPERTY` 为 C++类型定义属性，然后将其注册到 QML 类型系统中。

也可在 QML 文档的对象声明中使用以下方式直接定义属性。除了枚举之外，任何 QML 类型都可以作为属性类型。

```qml
[default] [required] [readonly] property <PropertyType> <propertyName>
```

- default，参下。

- required，创建对象实例时必须设置该属性。required 属性无法分配默认值。

- readonly，初始化时必须为其分配静态值或绑定到表达式，此后无法再修改该属性。

定义属性时，会隐式为该属性创建一个值更改的信号 `<propertyName>Changed`，以及名为 `on<PropertyName>Changed` 的信号处理程序。

<div class="code_block">

<div>
MyRectangle.qml

```qml
import QtQuick
import QtQuick.Controls

Rectangle {
  property color mColor

  Button {
    text: "change color"
    onClicked: parent.mColor = Qt.rgba(Math.random(), Math.random(), Math.random(), 1)
  }

  onMColorChanged: color = mColor
}
```

</div>

<div>
main.qml

```qml
import QtQuick.Controls
import "."

ApplicationWindow {
  width: 400
  height: 400
  visible: true

  MyRectangle {
    width: 200
    height: 200
  }
}
```

</div>
</div>

### default property

每个类型只能定义一个 default property，在对象内部定义其他对象时，如果没有明确指明其要分配到的属性，那么该对象就分配到 default property。

如 `Item` 定义了 default property 属性 `data`，其类型为 `list<QtObject>`，因此在 `Item` 类型中可以任意定义子对象。

> 如果继承自 `Item`，则将其作为子项，否则，作为资源项。

<div class="code_block">

<div>
示例：

```qml
ApplicationWindow {
  Item {
    TextInput {}

    Text {}

    QtObject {}

    Component.onCompleted: {
      console.log(data);
      console.log(children);
      console.log(resources);
    }
  }
}
```

</div>
<div>
输出：

```shell
qml: [QObject(0x560be37bb290),QQuickTextInput(0x560be3b30800),QQuickText(0x560be36b81d0)]
qml: [QQuickTextInput(0x560be3b30800),QQuickText(0x560be36b81d0)]
qml: [QObject(0x560be37bb290)]
```

</div>

</div>

```shell
qml: [QObject(0x558d71533480),QQuickTextInput(0x558d70acb380),QQuickText(0x558d70add3f0)]
qml: [QQuickTextInput(0x558d70acb380),QQuickText(0x558d70add3f0)]
qml: [QObject(0x558d71533480)]
```

子类定义的 default property 会覆盖父类定义的 default property。

### property alias

别名保存对另一个属性的引用，属性别名的定义语法为：

```qml
[default] property alias <name>: <reference>
```

别名存在以下限制：

- 只能引用类型范围内的对象或对象的属性。

- 不能包含 JS 表达式。

- 不能引用附加属性。

- 最多引用两层深度的属性别名。

别名可以和现有属性同名，并覆盖现有属性，此后无法通过属性名访问原有的属性，只能通过原有属性的引用访问。

## signal

在 C++中可以通过 `Q_SIGNAL` 定义一个信号，然后注册到 QML 类型系统中。

QML 中，使用以下语法定义信号：

```qml
signal <signalName> [([parameterName: parameterType, ...])]
```

如果信号没有参数，那么 `()` 是可选的，如果有参数，必须同时声明参数名和参数类型。如：

```qml
Item {
  signal clicked
  signal hovered
  signal checked(flag: bool)
}
```

## signal handler

信号处理程序是特殊的 method attribute，发出相关信号时，调用该方法。

<div class="code_block">

```qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
  width: 400
  height: 400
  visible: true

  Item {
    signal activated(x: double, y: double)
    signal deactivated

    width: parent.width
    height: parent.height

    MouseArea {
      anchors.fill: parent
      onReleased: parent.deactivated()
      onPressed: mouse => parent.activated(mouse.x, mouse.y)
    }

    onActivated: (x, y) => console.log(`activated at (${x}, ${y})`)
    onDeactivated: console.log("deactivated")
  }
}
```

</div>

## method

在 C++ 中可以将成员函数标记为 `Q_INVOKABLE` 或者 `Q_SLOT`，然后注册到 QML 类型系统中。

QML 使用以下语法定义方法：

```qml
function <funcName>([<parameterName>[: parameterType], ...])[: retType] { <body> }
```

方法必须要声明在对象内部。与 signal 不同，方法的参数类型可以不必声明，默认为 `var`。

在同一类型中定义同名的方法或信号是错误的，但可以定义同名的方法或信号覆盖父类的方法或信号。

## attached properties and attached signal handlers

通过 C++ 可以创建具有附加属性和信号的附加类型，在创建此类型示例时，就可以将附加属性和信号附加到特定对象上，从而实现非入侵式为其他对象附加新属性和信号。

如 `ListView` 具有附加属性 `.isCurrentItem`，附加在所有代理对象上，表示是否是当前选定项。

```qml
ListView {
    id: root
    width: 300
    height: 300
    focus: true
    model: ["item1", "item2", "item3"]
    delegate: Rectangle {
        width: parent.width
        height: 100
        color: ListView.isCurrentItem ? "gray" : "lightgray"
        Text {
            anchors.centerIn: parent
            text: modelData
        }
    }
}
```

## enumeration

枚举类型的声明语法如下，其中枚举类型必须以大写字母开头。

```js
enum EnumType { [EnumValue, ...] }
```

使用 `<EnumType>.<EnumValue>` 访问值。
