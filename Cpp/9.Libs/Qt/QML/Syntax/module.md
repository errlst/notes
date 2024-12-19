- [moduel definition files](#moduel-definition-files)
  - [module identifier declaration](#module-identifier-declaration)
  - [object type declaration](#object-type-declaration)
  - [internal object type declaration](#internal-object-type-declaration)

QML 模块通过 qmldir 文件声明，QML 提供两种类型的 qmldir 文件：

- document directory listing files。

- moduel definition files。

# moduel definition files

qmldir 是包含以下内容的纯文本文件：

- module identifier declaration，模块标识符声明。

- object type declaration，对象类型声明。

- internal object type declaration，内部对象类型声明。

- javascript resource declaration，JS 资源声明。

- plugin declaration，插件声明。

- plugin classname declaration，插件类名声明。

- type description file declaration，类型描述文件声明。

- module dependencies declaration，模块依赖声明。

- module import declaration，模块导入声明。

- designer suppore declaration。

- preferred path declaration，首选路径声明。

使用 `#` 符号作为注释起始。

## module identifier declaration

```qml
module <ModuleIdentifier>
```

模块标识符声明必须是 qmldir 文件的第一行，且一个 qmldir 中只能存在一个模块标识符声明。

## object type declaration

```qml
[singleton] <TypeName> <Version> <File>
```

声明模块可用的 QML 对象类型。

- singleton，声明单例类型。

- Version，使得该类型可用的模块版本。

- File，类型对应的 QML 文件。

<div style="display: flex; gap: 10px">

```qml
/// ./MyModule/MyStyle.qml
pragma Singleton
import QtQuick

QtObject {
    property int text_size: 40
    property color text_color: "lightblue"
}
```

```qml
/// ./MyModule/qmldir
module MyModule
singleton MyStyle MyStyle.qml
```

```qml
/// ./main.qml
import QtQuick
import QtQuick.Controls
import MyModule

ApplicationWindow {
    visible: true
    Text {
        text: "Hello World!"
        font.pixelSize: MyStyle.text_size
        color: MyStyle.text_color
    }
}
```

</div>

```shell
$ qml -I ./ ./main.qml
```

## internal object type declaration

内部对象在模块内部可使用，但不提供给使用模块的用户。

```qml
internal <TypeName> <File>
```
