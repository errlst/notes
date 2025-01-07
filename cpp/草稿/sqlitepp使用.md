#### 创建数据库连接

###### 构造时连接

如果连接失败，抛出 `sqlite3pp::database_error`。

```cpp
auto db = sqlite3pp::database{"", SQLITE_OPEN_READWRITE};
```

###### 手动连接

```cpp
// connect 的返回值为 sqlite3 capi 的返回值
auto db = sqlite3pp::database{};
db.connect("", SQLITE_OPEN_READWRITE);
```

#### 执行命令

多个线程在同一个 db 对象上执行命令是安全的。

可以直接使用 `db.execute()`，也可以创建 `command` 对象执行命令。对于数据的增删改使用 `command` 对象进行命令操作更合适。

###### 基本使用

```cpp
auto cmd = sqlite3pp::command{db, "create table test(id int)"};
cmd.execute();
```

###### 绑定参数

命令字符串中可以使用 `?`，之后可以使用其他参数替换 `?`。

```cpp
// 可以使用 binder() 顺序替换参数
{
	auto cmd = sqlite3pp::command{db, "select * from test where id = ?"};
	cmd.binder() << 333;
}
// 也可以使用 bind(idx, val) 指定绑定顺序，从 1 开始计数
{
    auto cmd = sqlite3pp::command{db, "select * from test where id = ?"};
    cmd.bind(1, 333);
}
```
