[type_index]()是对[type_info]()对象的封装，使其可以作为关联容器的索引使用。[示例](#示例1)[实现](#实现1)

# 示例

## 示例1

```cpp
auto main() -> int {
    auto m = std::map<std::type_index, std::string>{
        {typeid(int), "int"},
        {typeid(double), "double"}
    };
    for (auto &[k, v] : m) {
        std::cout << std::format("{} : {}\n", k.name(), v);
    }

    return 0;
}
```

# 实现

## 实现1

```cpp
class type_index {
  public:
    type_index(const std::type_info &info) noexcept: m_info{&info} {}

    auto operator<=>(const type_index &rhs) const noexcept -> std::strong_ordering {
        if (*m_info == *rhs.m_info) {
            return std::strong_ordering::equal;
        } else if (m_info->before(*rhs.m_info)) {
            return std::strong_ordering::less;
        }
        return std::strong_ordering::greater;
    }

    auto hash_code() const noexcept -> size_t { return m_info->hash_code(); }

    auto name() const noexcept -> const char * { return m_info->name(); }

  private:
    const std::type_info *m_info;
};

template<>
struct std::hash<type_index> {
    auto operator()(const type_index &i) const noexcept -> size_t { return i.hash_code(); }
};
```

