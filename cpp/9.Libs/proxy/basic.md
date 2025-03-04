```cpp
struct formatable
  : pro::facade_builder
  ::support_format
  ::build {};

auto main() -> int {
  auto p = pro::proxy<formatable>{};

  auto s = "hello";
  p = &s;
  std::println("{}", *p);

  p = std::make_unique<int>(123);
  std::println("{}", *p);

  p = pro::make_proxy<formatable>(123);
  std::println("{}", *p);

  return 0;
}
```

- `pro::facade_builder`，提供构建一个 facade 类型的能力。

- `support_format`，proxy 预定义的 convention。

- `build`，构建 facade 类型。

`proxy` 的行为类似于智能指针，当存储小对象时（小于指针大小），直接在原地存放。

---

```cpp
struct streamable
  : pro::facade_builder
  ::add_convention<
      pro::operator_dispatch<"<<", true>,
      std::ostream &(std::ostream &out)const>
  ::build {};

auto main() -> int {
  auto p = pro::proxy<streamable>{};

  auto s = std::string{"hello"};
  p = &s;
  std::cout << *p << "\n";

  p = std::make_unique<int>(123);
  std::cout << *p << "\n";

  p = pro::make_proxy<streamable>(123);
  std::cout << *p << "\n";

  return 0;
}
```

- `add_convention`，添加一个通用的调用约定，由一个 dispatch 和多个 overloads 定义。

- `pro::operator_dispatch<"<<", true>`，为运算符 `<<` 表达式指定 dispatch，其中 proxy 对象会位于右侧（由 `true` 指定）。

- `std::ostream &(std::ostream &out)const>`，指定函数签名。
