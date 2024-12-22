nginx 的队列是侵入式的双向链表，且头节点不会连接到数据节点上。

```c
struct ngx_queue_s {
    ngx_queue_t *prev;
    ngx_queue_t *next;
};
```

## 初始化

- `ngx_queue_init(q)`，初始化队列。

## 插入元素

- `ngx_queue_insert_head(q, v)`，头插元素。

- `ngx_queue_insert_tail(q, v)`，尾插元素。

## 遍历

- `ngx_queue_sentinel(q)`，哨兵位置（也就是头节点）。

- `ngx_queue_next(q_node)`，下一个节点。

- `ngx_queue_prev(q_node)`，前一个节点。

- `ngx_queue_data(q_node, type, link)`，访问节点上的元素。

  > ```c
  > #define ngx_queue_data(q, type, link) (type *)((u_char *)q - offsetof(type, link))
  > ```
  >
  > - `type`，侵入的类型。
  >
  > - `link`，侵入类型中 `ngx_queue_t` 变量的名字。

## 删除元素

- `ngx_queue_remove(q_node)`，移除当前节点。

## 示例

```c
typedef struct {
    ngx_int_t value;
    ngx_queue_t queue;
} queued_value;

void iter_queue(ngx_queue_t *queue) {
    ngx_queue_t *q = ngx_queue_head(queue);
    while (q != ngx_queue_sentinel(queue)) {
        queued_value *v = ngx_queue_data(q, queued_value, queue);
        printf("%lu ", v->value);
        q = ngx_queue_next(q);
    }
    printf("\n");
}

void test_queue(ngx_log_t *log) {
    ngx_pool_t *pool = ngx_create_pool(1024, log);

    ngx_queue_t queue;
    ngx_queue_init(&queue);

    for (ngx_int_t i = 0; i < 5; ++i) {
        queued_value *v = ngx_pcalloc(pool, sizeof(queued_value));
        v->value = i;
        ngx_queue_insert_tail(&queue, &v->queue);
    }

    iter_queue(&queue);
}
```
