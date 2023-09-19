[constinit]()声明的变量必须拥有[静态初始化]()[^2]。[示例]()

具有[constexpr]()构造函数但没有[constexpr]()析构函数的对象可以使用[constinit]()声明，但不能使用[constexpr]()声明。



# 示例

###### 示例1

```cpp
struct T {
	T() = default;
};

struct U {
	constexpr U() {}
};

constinit T t;		// ok，零初始化
constinit U u{};	// ok，常量初始化
```



[^1]:[零初始化]()和[常量初始化]()

