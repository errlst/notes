HTTP 客户端连接会经过以下过程：

- `ngx_event_accept()` 处理 tcp 连接，创建 `ngx_http_connection_t` 对象。

- `ngx_http_init_connection()`，处理 http 连接。

- `ngx_http_wait_request_handler()`，套接字可用时调用，创建 `ngx_http_request_t` 对象。

- `ngx_http_process_request_line()`，处理 http 请求行，具体解析函数为 `ngx_http_parse_request_line()`。

- `ngx_http_process_request_headers()`，处理 http 请求头，具体解析函数为 `ngx_http_parse_header_line()`。

- `ngx_http_core_run_phases()`，请求头解析完后，处理请求。

- `ngx_http_finalize_connection()`，响应发送到客户端后调用，调用 `ngx_http_close_request()` 销毁请求或调用 `ngx_http_set_keepalive()` 保持长连接。

## ngx_http_connection_t

