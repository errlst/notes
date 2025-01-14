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
