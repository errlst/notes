# 导入

## 语法

QML 文档有以下几种导入语法：

- `import <Module> <Version> <as Namespace>`，模块导入，指定版本，设置别名。

  - Module 是使用 URI 点分表示法的标识符，其唯一标识模块的命名空间。

  - Version 是 `Major.Minor` 形式的版本标识，Qt6 开始可以忽略，默认导入最新版本。

  - Namespace 为导入的模块指定命名空间，如果省略，模块直接导入到全局命名空间中。可以将多个模块导入到同一个命名空间中。命名空间必须以大写字母开头。可以将多个模块导入到同一个命名空间中。

- `import <Directory> <as Namespace>`，导入目录下的所有 QML 文档。目录甚至可以是远程 URL。

- `import <JSFile> <as Identifier>`，导入 js 文件。

  - Identifier 类似 Namespace，但在 QML 文档中必须唯一。

## 路径

QML 匹配搜索的模块的路径由 `QQmlEngine::importPathList()` 返回，默认情况该列表包含：

- 执行文件路径。

- `QLibraryInfo::QmlImportsPath` 指定的路径。

  > ```cpp
  > qDebug() << QLibraryInfo::path(QLibraryInfo::QmlImportsPath);
  > // out: /home/jsr/Qt/6.8.1/gcc_64/qml
  > ```

- 环境变量 `QML_IMPORT_PATH` 指定路径，在 Windows 平台通过 `;` 分割多个路径，其他平台通过 `:`。

- 资源路径 `qrc:/qt-project.org/imports`。

- 资源路径 `qrc:/qt/qml` (Qt6.5 开始)。
