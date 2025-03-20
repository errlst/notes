提供编译选项 -fno-access-control 可以关闭访问权限控制。

```cpp
class T {
  static auto call() -> void {}
};

auto main() -> int {
  T::call();
  return 0;
}
```

```shell
gcc main.cpp -fno-access-control
```
