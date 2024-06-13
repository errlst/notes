#### 规则

对于表达式 _e_，`decltype(e)` 表示的类型定义如下：

* 如果 _e_ 是带括号的实体表达式，则结果是左值引用。

  ```cpp
  auto main() -> int {
      int &&a = 0;
      std::cout << std::same_as<decltype(a), int &&> << "\n";
      std::cout << std::same_as<decltype((a)), int &> << "\n";
      return 0;
  }
  ```

* 如果 _e_ 是 _xvalue_，则结果是右值引用。

* 如果 _e_ 是 _lvalue_，则结果是左值引用。

  ```cpp
  auto main() -> int {
      auto a = " ";
      // 字符串常量是 lvalue，因此 decltye(" ") 是左值引用类型
      // a 通过类型推断，得到的是 const char* 类型
      std::cout << std::same_as<decltype(" "), decltype(a)>;
      return 0;
  }
  ```

* 否则，结果是 _e_ 的类型。

#### 编译器诊断信息

当某个类型导致编译失败时，报错信息一定会爆出该类型的具体类型，因此可以借助该特性，获取某个类型的详细信息。
```cpp
template<typename T>
class D;

auto main() -> int {
    int a = 10;
    int& r = a;
    int* p = &a;
    D<decltype(a)>(); // 使用未定义类型D<int>
    D<decltype(r)>(); // 使用未定义类型D<int&>
    D<decltype(p)>(); // 使用未定义类型D<int*>s
    return 0;
}
```