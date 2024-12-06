# 简介

`find_package` 的主要作用就是找到指定版本库的头文件路径、链接库路径等信息。

以链接 opencv 为例：

```cmake
cmake_minimum_required(VERSION 3.20)
project(cmake_demo)
find_package(OpenCV REQUIRED)

message(STATUS "OpenCV_DIR = ${OpenCV_DIR}")
message(STATUS "OpenCV_INCLUDE_DIRS = ${OpenCV_INCLUDE_DIRS}")
message(STATUS "OpenCV_LIBS = ${OpenCV_LIBS}")
```

> ```shell
> [cmake] -- OpenCV_DIR = '/usr/lib/x86_64-linux-gnu/cmake/opencv4'
> [cmake] -- OpenCV_INCLUDE_DIRS = '/usr/include/opencv4'
> [cmake] -- OpenCV_LIBS = 'opencv_calib3d;opencv_core;...'
> ```

`find_package` 本质上是通过一些特定规则找到 `<package_name>Config.cmake` 包配置文件，然后通过执行这些配置文件定义一系列变量，通过这些变量就可以准确定位到对应头文件和库文件，完成编译。

# 工作模式

`find_package` 有两种工作模式，这两种不同的工作模式决定其搜索路径：

- Module 模式。基础工作模式，也是默认模式。

- Config 模式。高级工作模式，只有指定了 `CONFIG`、`NO_MODULE` 等关键字，或者 Module 模式失败后使用该模式。

## Module

基本参数：

```cmake
find_package(<package> [version] [EXACT] [MODULE] [REQUIRED]
    [COMPONENTS [components...] ]
    ...
)
```

- EXACT。必须是指定版本，兼容版本也不行。

- MODULE。查找失败不会退回 Config 模式。

- REQUIRED。如果找不到包，立刻停掉整个 CMake。

- COMPONENTS, components...。查找的包中必须找到指定的组件。

### 搜索路径

Module 模式下搜索 `Find<PackageName>.cmake` 配置文件。只包含两个查找路径：`${CMAKE_MODULE_PATH}` 和 `${CMAKE_ROOT}/Modules` 目录。`CMAKE_MODULE_PATH` 默认为空，可以通过 `set` 赋值。

```cmake
cmake_minimum_required(VERSION 3.20)
project(cmake_demo)

message(STATUS "CMAKE_MODULE_PATH = '${CMAKE_MODULE_PATH}'")
message(STATUS "CMAKE_ROOT_PATH = '${CMAKE_ROOT}'")
```

> ```shell
> [cmake] -- CMAKE_MODULE_PATH = ''
> [cmake] -- CMAKE_ROOT_PATH = '/usr/share/cmake-3.22'
> ```

## Config

### 搜索路径

Config 模式搜索 `<PackageName>Config.cmake` 或 `<lower-case-package-name>-config.cmake` 配置文件。

具体查找顺序为：

1. 查找变量 `<PackageName>_DIR`。该路径需要指定到 `<PackageName>Config.cmake` 或 `<lower-case-package-name>-config.cmake` 文件所在目录才能找到包。

2. 查找变量 `CMAKE_PREFIX_PATH`、`CMAKE_FRAMEWORK_PATh`、`CMAKE_APPBUNDLE_PATH`。这些变量作为搜索的根目录。

3. 查找环境变量 `PATH`。遍历 `PATH` 中的各路径，如果以 `bin` 或 `sbin` 结尾，则退回到其上级目录并作为根目录进行以下规则的搜索：

   1. 搜索模块文件。

   2. 匹配以下路径作为根路径进行搜索：

   ```shell
   <prefix>/(lib/<arch> | lib | share)/cmake/<name>*/
   <prefix>/(lib/<arch> | lib | share)/<name>*/
   <prefix>/(lib/<arch> | lib | share)/<name>*/(cmake|CMake)/
   ```

   > 如通过 `/usr/bin` 找到 `/usr/lib/x86_64-linux-gnu/cmake`

> 以上的单独出现的 "变量" 表示环境变量或 CMake 变量。

# 指定包

如果明确知道需要查找库的 `<PackageName>.cmake` 或 `lower-case-package-name>-config.cmake` 所在路径，可以直接设置 `<PackageName>_DIR` 为具体路径。

也可以直接设置 `CMAKE_PREFIX_PATH` 等变量，这些变量默认都是空。

```cmake
set(CMAKE_PREFIX_PATH "/home/jsr/Qt/6.8.1/gcc_64/lib/cmake")
find_package(Qt6 REQUIRED COMPONENTS Core Quick QuickControls2)
target_link_libraries(exec PRIVATE Qt6::Core Qt6::Quick Qt6::QuickControls2)
```
