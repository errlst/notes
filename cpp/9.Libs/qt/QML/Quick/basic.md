QtQuick 是 Qt 提供的 QML 应用程序标准库，其提供了 QML API 和 C++ API，用于 QML 创建 UI 界面，和使用 C++代码拓展 QML 程序。

# 值类型

QtQuick 提供以下值类型：

| 类型                    | 说明                                        |
| ----------------------- | ------------------------------------------- |
| `color`                 | rgba 颜色                                   |
| `font`                  | 具有 `QFont` 的属性的值                     |
| `matrix4x4`             | 4x4 矩阵                                    |
| `quaternion`            | 四元数，具有 `scalar`、`x`、`y` 和 `z` 属性 |
| `vector2d` ~ `vector4d` | 2~4 维向量，具有 `x`、`y`、`z`、`w` 属性    |

# 对象类型

QtQuick 提供的大多数对象类型都是基于 `Item` 类型的，`Item` 类型从 `QtObject` 派生。
