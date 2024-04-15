#### 地址结构
套接字地址分为ipv4地址和ipv6地址，其结构名分别为 `sockaddr_in` 和 `sockaddr_in6`。

###### 通用地址结构
套接字接口在接受地址参数时，需要借助通用地址结构 `sockaddr`，以此保持接口的统一。posix标准规定该结构至少拥有以下两个成员：
```cpp
struct sockaddr {
    unsigned short sa_family;
    char sa_data[14];
};
```

同时，新标准还定义了新的通用地址结构 `sockaddr_storage`，其相比 `sockaddr` 有两个优势：
* 可以满足苛刻的对其要求。
* 足够大，能够容纳任何套接字地址结构。

###### ipv4地址结构
`sockaddr_in` 是ipv4套接字地址结构，`sin_family` 为 `AF_INET`。
```cpp
struct in_addr {
    uint32_t s_addr;
};

struct sockaddr_in {
    unsigned short sin_family;
    uint16_t sin_port; /* Port number.  */
    in_addr sin_addr;  /* Internet address.  */
    unsigned char sin_zero[sizeof(sockaddr) - sizeof(unsigned short) - 6];
};
```

###### ipv6地址结构
`sockaddr_in6` 是ipv6套接字地址结构，`sin_family` 为 `AF_INET6`。
```cpp
struct in6_addr {
    union {
        uint8_t __u6_addr8[16];
        uint16_t __u6_addr16[8];
        uint32_t __u6_addr32[4];
    } __in6_u;
};

struct sockaddr_in6 {
    unsigned short sin6_family;
    uint16_t sin6_port;     /* Transport layer port # */
    uint32_t sin6_flowinfo; /* IPv6 flow information */
    in6_addr sin6_addr;     /* IPv6 address */
    uint32_t sin6_scope_id; /* IPv6 scope-id */
};
```

###### 字节序转换
网络字节序采用大端方式对数据进行排序，因此标准提供了四个函数进行字节序之间的转换（主要用于端口的字节序转换）：
* `htons()`、`htonl()`，本地字节序转换到网络字节序。
* `ntohs()`、`ntohl()`，网络字节序转换到本地字节序。

###### 地址转换
