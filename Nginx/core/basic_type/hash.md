nginx 实现的哈希表支持两种查找方式：

- 精确匹配，在哈希表中查找同给定键完全相同的项。

- 通配符匹配，可以通过使用通配符（\*）匹配多个键，用于处理域名匹配的问题。

# 精确匹配

- `ngx_hash_ele_t`，哈希表中的一个元素。

  ```c
  typedef struct {
      void *value;
      u_short len;
      u_char name[1];
  } ngx_hash_elt_t;
  ```

  - `value`，存储的值。

  - `len`，键的长度。

  - `name`，键。

- `ngx_hash_key_t`，键值对。

  ```c
  typedef struct {
      ngx_str_t key;
      ngx_uint_t key_hash;
      void *value;
  } ngx_hash_key_t;
  ```

- `ngx_hash_t`，哈希表。

  ```c
  typedef struct {
      ngx_hash_elt_t **buckets;
      ngx_uint_t size;
  } ngx_hash_t;
  ```

  - `buckets`，哈希桶指针。

  - `size`，哈希桶数量。

- `ngx_hash_init_t`，初始化哈希表时需要的参数。

  ```cpp
  using ngx_hash_key_pt = ngx_uint_t (*)(u_char *data, size_t len);

  typedef struct {
      ngx_hash_t *hash;
      ngx_hash_key_pt key;
      ngx_uint_t max_size;
      ngx_uint_t bucket_size;
      char *name;
      ngx_pool_t *pool;
      ngx_pool_t *temp_pool;
  } ngx_hash_init_t;
  ```

  - `key`，哈希函数。

  - `max_size`，哈希表最大大小。

  - `bucket_size`，每个哈希桶的大小。

  - `name`，哈希表名。

  - `pool`，持久内存池。

  - `temp_pool`，临时内存池。

## 初始化

- `ngx_hash_init(hinit, names, nelts)`，初始化哈希表。

  - `names`，初始化时插入的键值对数组。

  - `nelts`，数组大小。
