- [id attribute](#id-attribute)
- [property attributes](#property-attributes)
  - [default property](#default-property)
  - [property alias](#property-alias)
- [signal attribute](#signal-attribute)
- [signal handler attribute](#signal-handler-attribute)
- [method attribute](#method-attribute)
- [attached properties and attached signal handlers](#attached-properties-and-attached-signal-handlers)
- [enumeration attribute](#enumeration-attribute)

---

QML 对象可以包含以下属性：

- id attribute。

- property attributes。

- signal attributes。

- method attributes。

- attached properties and attached signal handlers attributes。

- enumeration attributes。

# id attribute

id 属性由 QML 语言自身提供，每个 QML 对象只能有一个 id 属性，且必须唯一。创建对象示例后就无法修改 id 属性，且 id 并非普通的属性，无法适用于某些语法：如 `.id` 是非法的。

id 值必须以小写字母或下划线开头，且只能由字母、数字或下划线组成。

> ```js
> Column {
>     width: 200
>     height: 200
>
>     TextInput {
>         id: text_input
>         text: "text_input"
>     }
>
>     Text {
>         text: text_input.text
>     }
> }
> ```

# property attributes

可以为 property 分配静态值或绑定到表达式。

在 C++中，可以通过 `Q_PROPERTY` 为 C++类型定义属性，然后将其注册到 QML 类型系统中。

也可在 QML 文档的对象声明中使用以下方式直接定义属性。除了枚举之外，任何 QML 类型都可以作为属性类型。

> ```js
> [default] [required] [readonly] property <PropertyType> <propertyName>
> ```
>
> - default，参下。
>
> - required，创建对象实例时必须设置该属性。required 属性无法分配默认值。
>
> - readonly，初始化时必须为其分配静态值或绑定到表达式，此后无法再修改该属性。

定义属性时，会隐式为该属性创建一个值更改的信号 `propertyNameChanged`，以及名为 `on<PropertyName>Changed` 的信号处理程序。

> ```js
> // MyRectangle.qml
> import QtQuick
> import QtQuick.Controls
>
> Rectangle {
>     property color mColor
>
>     Button {
>         onClicked: parent.mColor = Qt.rgba(Math.random(), Math.random(), Math.random(), 1)
>     }
>
>     onMColorChanged: color = mColor
> }
>
> // main.qml
> import "."
>
> MyRectangle {}
> ```

## default property

每个类型只能定义一个 default property，当定义对象时，如果其子对象没有明确指明其要分配到的属性名，那么该子对象就分配到 default property。

如 `Item` 定义了 default property 属性 `data`，其类型为 `list<QtObject>`，因此在 `Item` 类型中可以任意定义子对象，如果添加 `Item` 项，则将其作为子项，否则，作为资源项。

> ```js
> Item {
>     TextInput {}
>
>     Text {}
>
>     QtObject {}
>
>     Component.onCompleted: {
>         console.log(data);
>         console.log(children);
>         console.log(resources);
>     }
> }
> ```
>
> ```shell
> qml: [QObject(0x558d71533480),QQuickTextInput(0x558d70acb380),QQuickText(0x558d70add3f0)]
> qml: [QQuickTextInput(0x558d70acb380),QQuickText(0x558d70add3f0)]
> qml: [QObject(0x558d71533480)]
> ```

子类定义的 default property 会覆盖父类定义的 default property。

## property alias

别名保存对另一个属性的引用，属性别民的定义语法为：

```js
[default] property alias <name>: <reference>
```

别名存在以下限制：

- 只能引用类型范围内的对象或对象的属性。

- 不能包含 JS 表达式。

- 不能引用附加属性。

- 最多引用两层深度的属性别名。

别名可以和现有属性同名，并覆盖现有属性，此后无法通过属性名访问原有的属性，只能通过原有属性的引用访问。

# signal attribute

在 C++中可以通过 `Q_SIGNAL` 定义一个信号，然后注册到 QML 类型系统中。

QML 中，使用以下语法定义信号：

> ```js
> signal <signalName> [([parameterName: parameterType, ...])]
> ```
>
> 如果信号没有参数，那么 `()` 是可选的，如果有参数，必须同时声明参数名和参数类型。如：
>
> ```js
> Item {
>     signal clicked
>     signal hovered
>     signal checked(flag: bool)
> }
> ```
>
> 同函数调用一样，即发出信号。

# signal handler attribute

信号处理程序是特殊的 method attribute，当发出相关信号是，都会调用该方法。

```js
Item {
    id: root

    signal activated(x: double, y: double)
    signal deactivated

    width: 200
    height: 200

    MouseArea {
        anchors.fill: parent
        onReleased: root.deactivated()
        onPressed: mouse => root.activated(mouse.x, mouse.y)
    }

    onActivated: (x, y) => console.log(`activated at (${x}, ${y})`)
    onDeactivated: console.log("deactivated")
}
```

# method attribute

在 C++中可以将成员函数标记为 `Q_INVOKABLE` 或者 `Q_SLOT`，然后注册到 QML 类型系统中。

QML 使用以下语法定义方法：

```js
function <funcName>([<parameterName>[: parameterType], ...])[: retType] { <body> }
```

方法必须要声明在对象内部。与 signal 不同，方法的参数类型可以不必声明，默认为 `var`。

在同一类型中定义同名的方法或信号是错误的，但可以定义同名的方法或信号覆盖父类的方法或信号。

# attached properties and attached signal handlers

通过 C++ 可以创建具有附加属性和信号的附加类型，在创建此类型示例时，就可以将附加属性和信号附加到特定对象上，从而实现非入侵式为其他对象附加新属性和信号。

如 `ListView` 具有附加属性 `.isCurrentItem`，附加在所有代理对象上，表示是否是当前选定项。

```js
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

# enumeration attribute

枚举类型的声明语法如下，其中枚举类型必须以大写字母开头。

```js
enum EnumType { [EnumValue, ...] }
```

使用 `<EnumType>.<EnumValue>` 访问值。
