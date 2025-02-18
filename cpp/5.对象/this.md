# deducing this

C++23 引入，可以将 `this` 通过参数显示传递到成员函数中。

## 简化成员函数重载

C++23 前：

```cpp
template <typename T>
struct optional {
  auto value() & -> T & {}
  auto value() const & -> const T & {}
  auto value() && -> T && {}
  auto value() const && -> const T && {}
};
```

C++23 后：

```cpp
template <typename T>
struct optional {
  auto &&value(this auto &&self) {
    return std::forward<Self>(self).val_;
  }

  T val_;
};
```

## lambda 递归

```cpp
auto fiber = [](this auto &&self, int n) {
  if (n <= 2) {
    return 1;
  }
  return self(n - 1) + self(n - 2);
};
```

## 简化 CRTP

C++23 前：

```cpp
template <typename Derive> struct add_postfix_increment {
  auto operator++(int) {
    auto derive = reinterpret_cast<Derive *>(this);
    auto ret = *derive;
    ++(*derive);
    return ret;
  }
};

struct some_type : public add_postfix_increment<some_type> {
  using add_postfix_increment<some_type>::operator++;
  auto operator++() { return *this; }
};
```

C++23 后：

```cpp
struct add_postfix_increment {
  auto operator++(this auto &&self, int) {
    auto ret = self;
    ++self;
    return ret;
  }
};

struct some_type : add_postfix_increment {
  using add_postfix_increment::operator++;
  auto operator++() { return *this; }
};
```
