[toc]

[array]()是封装固定大小数组的容器，其基本结构如下：

```cpp
template<typename _T, size_t N>
struct array {
    _T m_elements[N];   // 需要对零长数组进行特化处理：
                        // libstdc++ --> 通过『类型萃取』特化元素的类型
                        // mscv --> 直接特化『array<T, 0>』

    // 通过『推导指南』，根据传入的参数自动推断数组容量
    template<typename T, typename ... Ts>
    array(T, Ts...) ->
    array<std::enable_if_t<(std::is_same_v<T, Ts> && ...), T>,  // 确保元素类型相同
          1 + sizeof...(Ts)>;
};
```

## 创建数组

除了直接通过元素创建[array]()，还可以使用[to_array()]()将内置数组转换为[array]()对象。

## 元组接口

[array]()满足元组接口。

## 零长尺寸

libstdc++中，零长[array]()的尺寸为1字节。

msvc中，零长[array]()的尺寸为『元素大小字节』，其对[array<T, 0>]()进行了特化，但依然保留的一个元素。





