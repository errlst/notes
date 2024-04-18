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