`case` 实际上就是给一段语句添加一个标签，在 `switch` 时，会根据表达式直接跳转到对应语句的起始位置，然后执行。

```cpp
auto main() -> int {
    int i = 1;
    switch (i) {
        case 0:
            while (i == 1) {
                std::cout << "1\n";
                case 1:
                    std::cout << "2\n";        // 从此条语句开始执行
                    break;                     // 离开 while
            }
        case 2:
            std::cout << "3\n";                // 下落到此处
        case 3:
            break;                             // 离开 switch
            std::cout << "4\n";                // 不会执行
    }
    return 0;
}
// 输出：23
```

