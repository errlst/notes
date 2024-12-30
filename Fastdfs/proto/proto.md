fastdfs 协议相关内容定义在 `tracker/tracker_proto.h` 文件。

## 协议头部

```c
typedef struct
{
	char pkg_len[FDFS_PROTO_PKG_LEN_SIZE];  // payload 长度，不包含 header
	char cmd;    // 指令
	char status; // 响应状态
} TrackerHeader;
```

- cmd，协议指令定义为系列宏 `TRACKER_PROTO_CMD_XXX`。

## 获取 storage 地址

此协议只在 client <-> tracker 有效。

```c
// 获取单个 storage 地址
#define TRACKER_PROTO_CMD_SERVICE_QUERY_STORE_WITHOUT_GROUP_ONE 101

// 获取指定组中单个 storage 地址
#define TRACKER_PROTO_CMD_SERVICE_QUERY_STORE_WITH_GROUP_ONE 104

// 查询所有 storage 地址
#define TRACKER_PROTO_CMD_SERVICE_QUERY_STORE_WITHOUT_GROUP_ALL 106

// 查询指定组中所有 storage 地址
#define TRACKER_PROTO_CMD_SERVICE_QUERY_STORE_WITH_GROUP_ALL 107
```

查询单个 storage 的完整响应为：`| header | group_name | ip_addr | port |`，payload 长度固定 `TRACKER_QUERY_STORAGE_STORE_BODY_LEN`。

- group_name，长度固定为 `FDFS_GROUP_NAME_MAX_LEN`。

- ip_addr，长度固定为 `IP_ADDRESS_SIZE - 1`，同时支持 ipv4 和 ipv6。

> port 后还预留了 `FDFS_PROTO_PKG_LEN_SIZE - 2` 大小的空间。

查询所有 storage 的完整响应为：`| header | group_name | ip_addr_and_port_list |`。

- group_name，如果没有指定组，则该值忽略（依然占据内存）。

- ip_addr_and_port_list，每个 ip-port 长度为 `RECORD_LENGTH`。

## 常用 api

### upload & upload_appender

upload，上传普通文件，包括主文件。

```c
#define STORAGE_PROTO_CMD_UPLOAD_FILE 11
```

upload_appender，上传可追加文件，后续可对文件进行 append、modify 和 trunc 操作。

```c
#define STORAGE_PROTO_CMD_UPLOAD_APPENDER_FILE	23
```

两个 api 的 payload 格式相同：

```text
| store_idx | size | ext | content |
```

- store_index，1 字节，storage 的存储索引，用于负载均衡。

> 该参数对应 storage 配置文件中的 `store_pathn`，从 tracker 获取。

- size，8 字节，文件大小。

- ext，`FDFS_FILE_EXT_NAME_MAX_LEN` 字节，文件后缀（不包括 `.`）。

- content，文件内容。

### upload_slave

从文件基于主文件进行命名，常用于缩略图。

```c
#define STORAGE_PROTO_CMD_UPLOAD_SLAVE_FILE	21
```

协议 payload 格式为：

```text
| master_len | size | prefix | ext | master_name | content |
```

- master_len，8 字节，主文件名长度。

- prefix，从文件前缀。

- master_name，主文件名。
