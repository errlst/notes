# 字符串

nginx 使用 `u_char*` 表示 C 字符串。nginx 的字符串类型定义如下：

```c
// src/core/ngx_string.h
typedef struct {
    size_t len;
    u_char *data;
} ngx_str_t;
```

字符串的长度由 `len` 定义，`data` 的末尾可能以空结尾，也可能以非空结尾。
