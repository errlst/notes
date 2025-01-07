# 本地打包

`xmake package`，使用 `package()` 的方式描述本地包，使得可以和远程包的方式一样集成本地包，会在 `build/packages` 下生成 xmake.lua 文件。

## 示例

两个项目：binary 可执行文件，static_package 静态库。

```shell
.
├── binary
│   ├── build
│   │   └── linux
│   ├── src
│   │   └── main.cpp
│   └── xmake.lua
└── static_package
    ├── build
    │   ├── linux
    │   └── packages
    ├── src
    │   ├── foo.cpp
    │   ├── foo.h
    │   └── main.cpp
    └── xmake.lua
```

### static_packge

```shell
$ xmake create -t static -P ./static_package

$ xmake package foo
```

> 生成的 xmake.lua 中还需要手动加上 `add_headerfiles("src/foo.h")`。

### binary

```shell
$ xmake create -t conosle -P ./binary
```

```lua
add_rules("mode.debug", "mode.release")

-- 添加本地仓库目录
add_repositories("local-repo ../static_package/build")
add_requires("foo")

target("binary")
    set_kind("binary")
    add_files("src/*.cpp")
    add_packages("foo")
```

# 远程打包

`xmake package -f remote` 生成远程包，远程包和本地包的主要区别是远程包多了 urls 设置和安装逻辑。

# 从 cmake 搜索包

