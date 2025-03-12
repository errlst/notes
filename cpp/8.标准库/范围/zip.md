<link href="../../..//style.css" rel="stylesheet">

# zip_view

```cpp
template <input_range... _Views>
  requires(view<_Views> && ...) && (sizeof...(_Views) > 0)
class zip_view : public view_interface<zip_view<_Views...>>;
```

`zip_view` 接受一个或多个视图，产生的新的视图的第 i 个元素由视图的第 i 个元素组成的元组，生成的视图的大小是所有视图的最小大小。

<div class="example_block">

<div class="example_block_code">

```cpp
auto main() -> int {
  auto vs = std::vector{1, 2, 3};
  auto ss = {"壹", "贰"};

  for (const auto &[v, s] : std::ranges::views::zip(vs, ss)) {
    std::println("{} {}", v, s);
  }

  return 0;
}
```

</div>

<div class="example_block_output">

```shell
1 壹
2 贰
```

</div>
</div>

# zip_transform_view
