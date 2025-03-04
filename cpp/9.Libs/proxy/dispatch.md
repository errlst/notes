proxy 提供了以下几种方式定义 dispatch。

## PRO_DEF_MEM_DISPATCH

定义一个成员函数表达式的 dispatch。提供两种语法：

- `PRO_DEF_FREE_DISPATCH(dispatch_name, func_name);`。

  ```cpp
  PRO_DEF_MEM_DISPATCH(public_callable_dispatch, call);

  struct public_callable
    : pro::facade_builder
    ::add_convention<public_callable_dispatch, void()>
    ::build {};

  struct foo {
    auto call() -> void {}
  };

  auto main() -> int {
    auto p = pro::make_proxy<public_callable>(foo{});
    p->call();
    return 0;
  }
  ```

- `PRO_DEF_FREE_DISPATCH(dispatch_name, func_name, accessibility_func_name);`。通过 `accessibility_func_name` 代理调用 `func_name`。

  ```cpp
  PRO_DEF_MEM_DISPATCH(public_callable_dispatch, call, CALL);

  struct public_callable
    : pro::facade_builder
    ::add_convention<public_callable_dispatch, void()>
    ::build {};

  struct foo {
    auto call() -> void {}
  };

  auto main() -> int {
    auto p = pro::make_proxy<public_callable>(foo{});
    p->CALL();
    return 0;
  }
  ```

## PRO_DEF_FREE_DISPATCH

定义一个自由函数表达式的 dispatch。同样提供两种语法。

- `PRO_DEF_FREE_DISPATCH(dispatch_name, func_name);`

- `PRO_DEF_FREE_DISPATCH(dispatch_name, func_name, accessibility_func_name);`。

```cpp
PRO_DEF_FREE_DISPATCH(stringable_dispatch, std::to_string, tostring);

struct stringable
  : pro::facade_builder
  ::add_convention<stringable_dispatch, std::string() const>
  ::build {};

auto main() -> int {
  auto p = pro::make_proxy<stringable>(1);
  tostring(*p);
  return 0;
}
```

## PRO_DEF_FREE_AS_MEM_DISPATCH

定义一个自由函数表达式的 dispatch，但通过成员函数的方式调用，提供两种语法。

- `PRO_DEF_FREE_AS_MEM_DISPATCH(dispatch_name, func_name);`。等价 `PRO_DEF_FREE_AS_MEM_DISPATCH(dispatch_name, func_name, func_name);`。

- `PRO_DEF_FREE_AS_MEM_DISPATCH(dispatch_name, func_name, accessibility_func_name);`。

```cpp
PRO_DEF_FREE_AS_MEM_DISPATCH(stringable_dispatch, std::to_string, tostring);

struct stringable
  : pro::facade_builder
  ::add_convention<stringable_dispatch, std::string() const>
  ::build {};

auto main() -> int {
  auto p = pro::make_proxy<stringable>(1);
  p->tostring();
  return 0;
}
```

## operator_dispatch

```cpp
template <details::sign Sign, bool Rhs = false>
struct operator_dispatch;
```

- Sign，运算符的字符串常量。

- Rhs，二元运算符中 proxy 对象是否位于运算符右侧。

```cpp
template <typename T> using prefix_inc_overload = pro::proxy<T> &();
template <typename T> using postfix_inc_overload = pro::proxy<T>(int);

struct prefix_incable
    : pro::facade_builder::add_convention<
          pro::operator_dispatch<"++">,
          pro::facade_aware_overload_t<prefix_inc_overload>>::build {};

struct postfix_incable
    : pro::facade_builder::add_convention<
          pro::operator_dispatch<"++">,
          pro::facade_aware_overload_t<postfix_inc_overload>>::build {};

struct prefix_inc {
  auto operator++() -> prefix_inc & { return *this; }
};

struct postfix_inc {
  auto operator++(int) -> postfix_inc { return *this; }
};

auto main() -> int {
  auto p1 = pro::make_proxy<prefix_incable>(prefix_inc{});
  auto p2 = pro::make_proxy<postfix_incable>(postfix_inc{});
  return 0;
}
```