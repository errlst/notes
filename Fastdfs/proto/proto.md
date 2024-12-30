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

## 上传文件

此协议只在 client <-> storage 有效。

```c
#define STORAGE_PROTO_CMD_UPLOAD_FILE 11
```

有两种上传协议格式：

- `| header | filename_len | file_size | filename | data |`。

  - filename_len，固定长度 `FDFS_PROTO_PKG_LEN_SIZE`。

  - file_size，固定长度 `FDFS_PROTO_PKG_LEN_SIZE`。

  - filename，固定长度 `FDFS_FILE_PREFIX_MAX_LEN + FDFS_FILE_EXT_NAME_MAX_LEN`。

- `| header | storage_idx | file_size | extname | data|`。

- storage_idx，向 tracker 申请 storage index 的结果，1 字节。

- extname，文件后缀名或全 0。
