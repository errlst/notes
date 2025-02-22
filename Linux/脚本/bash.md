不同的 shell 的脚本语法并不完全一致，在脚本第一行可加上 `#!/bin/bash` 指明该脚本使用 bash 解析器执行。

## 变量

定义变量语法为 `var=val`，变量和值之间不能存在空格。

除了显示的直接赋值变量，还可以用语句为变量赋值，如：

```shell
for file in $(ls); do
    echo $file
done
```

#### 使用变量

使用 `$val` 获取变量。也可加上额外的 `{}` 符号帮助解释器识别变量边界，如：

```shell
n=0
for file in $(ls); do
    echo "${n}: ${file}"
    n=$(expr $n + 1)
done
```

#### 只读变量

默认定义的变量是可以被重新赋值的，使用 `readonly` 命令将变量设置为只读变量。

```shell
n=1
echo $n # 1

n=2
echo $n # 2

readonly n
n=3     # ./a.sh: line 10: n: readonly variable
echo $n # 2
```

一旦将变量设为只读，就无法取消其只读性。

#### 删除变量

使用 `unset val` 删除变量，但不能删除只读变量。

#### 特殊变量

bash 脚本中具有一些特殊变量：

- `$0`，当前脚本名称。

- `$1`、`$2`、...，脚本参数。

- `$#`，参数数量。

- `$?`，上一个命令的退出状态。

## 字符串

字符串可以使用 `"`、`'` 或者不使用引号。

`'`字符串存在以下限制：

- 其中的所有字符都会原样输出，其中的变量是无效的。

- 字符串中不能存在 `'`，即使使用转义字符。

#### 字符串长度

当变量为字符串时，使用 `${#val}` 获取字符串长度。

```shell
str="hello"
echo ${#str}    # 5
```
