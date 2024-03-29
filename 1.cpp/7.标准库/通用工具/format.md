## 格式字符串

格式字符串中，包含普通字符、转义字符[^1]以及替换域。

### 替换域

替换域的基本格式为：`{ idx : spec }`。其中：

* _idx_指示当前替换域对应的参数索引，且自动索引和手动索引不能混合使用。[示例](#示例1)
* _spec_指示格式化规范：
  * 对于内置类型和标准字符串类型，格式说明为[标准格式说明]()。
  * 对于标准日期和时间类型，格式说明为[chrono格式说明]()。
  * 对于用户定义类型，格式说明由[std::formatter]()特化决定。

### 



# 示例

## 示例1

```cpp
auto main() -> int {
    std::cout << std::format("{1} {0}", 0, 1);  // 1 0
    std::cout << std::format("{0} {}", 0, 1);   // error

    return 0;
}
```







[^1]:`{{`转义为`{`，`}}`转义为`}`。