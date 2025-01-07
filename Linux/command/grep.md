```shell
grep [options...] patern [files...]
```

如果需要匹配多个 patern，使用 `-e`：

```shell
grep [options...] -e patern_0 -e patern_1 [files...]
```

`-i`，忽略大小写。

`-c`，统计匹配的行数量。

`-c <n>`，输出匹配行的前后 n 行。

`-o`，只输出匹配的部分。

`-r`，遍历目录及其子目录下所有文件。
