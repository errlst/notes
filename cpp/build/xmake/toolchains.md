xmake repo 仓库收录了一些工具链包，可以直接通过 xmake 远程拉取工具链管理。

## 拉取 llvm 工具链

```lua
add_rules("mode.debug", "mode.release")

add_requires("llvm 18.x", {alias="llvm-18"})

target("llvm-cc")
    set_kind("binary")
    add_files("src/*.cpp")
    set_languages("c++26")
    set_toolchains("llvm@llvm-18")
```

使用 llvm 工具链时，如果需要使用 libc++，除了添加 `-stdlib=libc++`，还需要添加链接选项 `-lc++`。

```lua
target("llvm-cc")
    set_kind("binary")
    add_files("src/*.cpp")
    add_cxflags("-stdlib=libc++")
    add_ldflags("-lc++")
    set_languages("c++26")
    set_toolchains("clang-20")
```
