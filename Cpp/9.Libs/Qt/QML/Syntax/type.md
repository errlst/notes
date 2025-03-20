- [值类型](#值类型)
  - [内置值类型](#内置值类型)
  - [属性更改](#属性更改)
- [对象类型](#对象类型)
  - [从 QML 定义对象类型](#从-qml-定义对象类型)
    - [从 QML 文档定义对象类型](#从-qml-文档定义对象类型)
    - [使用 Component 定义匿名类型](#使用-component-定义匿名类型)
  - [从 C++ 定义对象类型](#从-c-定义对象类型)
- [序列类型](#序列类型)
- [命名空间](#命名空间)
- [JS 类型](#js-类型)

# 值类型

值类型是在概念上通过值而不是引用传递的类型。将值的实例分配给两个属性，则这两个属性将带有单独的值，对属性的修改不会反应到值的实例上。

## 内置值类型

QML 内置以下值类型，不需要导入任何模块：

| 类型          | 说明                                       |
| ------------- | ------------------------------------------ |
| `bool`        | true 或 false                              |
| `date`        | 日期值，包含年月日、时分秒、毫秒、时区偏移 |
| `double`      | 双精度浮点                                 |
| `enumeration` | 命名枚举值                                 |
| `int`         | 4 字节有符号整数                           |
| `list`        | 列表                                       |
| `real`        | 双精度浮点                                 |
| `string`      | 字符串                                     |
| `url`         | 资源定位器                                 |
| `var`         | 泛型                                       |
| `variant`     | 泛型                                       |
| `void`        | 空值类型                                   |

## 属性更改

值类型也可以拥有属性，但其属性不具有属性更改信号，但值类型本身具有属性更改信号，当值类型的任何属性发生变化或者其本身发生变化后，都会触发该信号。

```qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    width: 400
    height: 400
    visible: true

    Rectangle {
        id: rect
        width: parent.width
        height: parent.height

        // color 是一个值类型
        onColorChanged: {
            console.log(`(${color.r}, ${color.g}, ${color.b})`);
        }
    }

    Button {
        onClicked: {
            rect.color.r = Math.random();
            rect.color.g = Math.random();
            rect.color.b = Math.random();
        }
    }
}
```

相反，对象类型的属性只有当自身发生变化时才会产生属性变化信号。

# 对象类型

QML 所有对象都派生自 `QtObject`，且必须以大写字母开头。

## 从 QML 定义对象类型

### 从 QML 文档定义对象类型

当一份 QML 文档对 QML 导入系统可见时，其定义了一个类型，如 `MyButton.qml` 定义了类型 `MyButton`。

### 使用 Component 定义匿名类型

在对象内部定义 `Component` 子对象，然后通过 `Component::createObject()` 实例化，实现在文档内部定义匿名类型并创建。

```qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    id: root
    width: 400
    height: 400
    visible: true

    Component {
        id: my_rectangle
        Rectangle {
            width: 100
            height: 100
            color: Qt.rgba(Math.random(), 1, 1, 1)
        }
    }

    Component.onCompleted: {
        my_rectangle.createObject(root);
        my_rectangle.createObject(root, {
            "x": 200
        });
    }
}
```

## 从 C++ 定义对象类型

# 序列类型

使用 `list<type>` 定义存储 `type` 实例的序列类型，使用 `[data1, ...]` 定义序列字面量。

> 值类型的序列的实现为 `QList`，对象类型的序列的实现为 `QQmlListProperty`。

QML 序列的通常行为表现类似与 JS 的 Array 类型，但存在以下差异：

- `delete` 某个元素会使用默认构造的值替换该元素，而不是 `undefined`。

  ```qml
  import QtQuick
  import QtQuick.Controls

  ApplicationWindow {
      id: root

      Component.onCompleted: {
          var js_list = [1, 2, 3];
          delete js_list[1];
          console.log(js_list[1]);    // undefined

          delete root.qml_list[1];
          console.log(root.qml_list[1]);  // 0
      }

      property list<int> qml_list: [1, 2, 3]
  }
  ```

- 将序列的 `length` 属性设置大于当前的值时，会使用默认构造的元素填充，而不是 `undefined`。

# 命名空间

# JS 类型
