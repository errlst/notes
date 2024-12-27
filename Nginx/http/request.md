# ngx_http_request_t

```c
struct ngx_http_request_s {
    uint32_t signature; // 帮助调试的签名，对应ascii "HTTP"

    // 客户端连接。
    // 多个请求可以同时引用一个连接，此时存在一个主请求和多个子请求，connection->data 指向主请求
    ngx_connection_t *connection;

    void **ctx;       // 请求上下文配置
    void **main_conf; // 主配置
    void **srv_conf;  // 服务器配置
    void **loc_conf;  // 位置配置

    // 读写处理函数
    ngx_http_event_handler_pt read_event_handler;
    ngx_http_event_handler_pt write_event_handler;

// 缓存上游请求
#if (NGX_HTTP_CACHE)
    ngx_http_cache_t *cache;
#endif

    // 上游服务器信息和状态
    ngx_http_upstream_t *upstream;
    ngx_array_t *upstream_states; // ngx_http_upstream_state_t 数组

    ngx_pool_t *pool;

    ngx_buf_t *header_in;                  // 请求头原始数据
    ngx_http_headers_in_t headers_in;      // 解析后的请求头
    ngx_http_headers_out_t headers_out;    // 需要发送的响应头
    ngx_http_request_body_t *request_body; // 请求体

    // 请求相关时间戳
    time_t lingering_time;
    time_t start_sec;
    ngx_msec_t start_msec;

    ngx_uint_t method;       // http 方法
    ngx_uint_t http_version; // http 版本
    ngx_str_t request_line;  // 原始请求行
    ngx_str_t uri;           // uri
    ngx_str_t args;          // 查询参数
    ngx_str_t exten;         // 文件拓展名
    ngx_str_t unparsed_uri;  // 未解析 uri

    ngx_str_t method_name;   // 方法字符串
    ngx_str_t http_protocol; // 版本字符串
    ngx_str_t schema;        // 方案字符串，如 "http"、"https"

    ngx_chain_t *out; // 输出缓冲链

    // 主请求是最初客户端发起并由 nginx 处理的请求，每个连接上只会有一个主请求
    //
    // 有些情况下，一个请求可能需要发起更多请求执行额外任务或资源，这种情况就会创建父子关系
    ngx_http_request_t *main;                    // 主请求
    ngx_http_request_t *parent;                  // 父请求
    ngx_http_postponed_request_t *postponed;     // 输出缓冲区和子请求列表
    ngx_http_post_subrequest_t *post_subrequest; // 子请求完成时需要使用的数据
    ngx_http_posted_request_t *posted_requests;

    ngx_int_t phase_handler; // 请求当前阶段
    ngx_http_handler_pt content_handler;
    ngx_uint_t access_code;

    ngx_http_variable_value_t *variables; // 动态变量数组

// PCRE 正则使用
#if (NGX_PCRE)
    ngx_uint_t ncaptures;
    int *captures;
    u_char *captures_data;
#endif

    // 速率限制和生效时间
    size_t limit_rate;
    size_t limit_rate_after;

    /* used to learn the Apache compatible response length without a header */
    size_t header_size;

    off_t request_length;

    ngx_uint_t err_status;

    ngx_http_connection_t *http_connection;
    ngx_http_v2_stream_t *stream;
    ngx_http_v3_parse_t *v3_parse;

    ngx_http_log_handler_pt log_handler;

    ngx_http_cleanup_t *cleanup;

    unsigned count : 16;      // 请求引用计数，仅对主请求有效
    unsigned subrequests : 8; // 子请求的嵌套级别，subrequests=parent->subrequests-1，主请求的值为 NGX_HTTP_MAX_SUBREQUESTS
    unsigned blocked : 8;     // 请求保留的块的数量，如果非 0，无法终止请求，aio 操作会增加此值

    unsigned aio : 1;

    unsigned http_state : 4;

    /* URI with "/." and on Win32 with "//" */
    unsigned complex_uri : 1;

    /* URI with "%" */
    unsigned quoted_uri : 1;

    /* URI with "+" */
    unsigned plus_in_uri : 1;

    /* URI with empty path */
    unsigned empty_path_in_uri : 1;

    unsigned invalid_header : 1;

    unsigned add_uri_to_alias : 1;
    unsigned valid_location : 1;
    unsigned valid_unparsed_uri : 1;
    unsigned uri_changed : 1;
    unsigned uri_changes : 4; // 剩余 uri 可修改次数，内部重定向、重写被认为是 uri 修改

    unsigned request_body_in_single_buf : 1;
    unsigned request_body_in_file_only : 1;
    unsigned request_body_in_persistent_file : 1;
    unsigned request_body_in_clean_file : 1;
    unsigned request_body_file_group_access : 1;
    unsigned request_body_file_log_level : 3;
    unsigned request_body_no_buffering : 1;

    unsigned subrequest_in_memory : 1;
    unsigned waited : 1;

#if (NGX_HTTP_CACHE)
    unsigned cached : 1;
#endif

#if (NGX_HTTP_GZIP)
    unsigned gzip_tested : 1;
    unsigned gzip_ok : 1;
    unsigned gzip_vary : 1;
#endif

#if (NGX_PCRE)
    unsigned realloc_captures : 1;
#endif

    unsigned proxy : 1;
    unsigned bypass_cache : 1;
    unsigned no_cache : 1;

    /*
     * instead of using the request context data in
     * ngx_http_limit_conn_module and ngx_http_limit_req_module
     * we use the bit fields in the request structure
     */
    unsigned limit_conn_status : 2;
    unsigned limit_req_status : 3;

    unsigned limit_rate_set : 1;
    unsigned limit_rate_after_set : 1;

#if 0
    unsigned                          cacheable:1;
#endif

    unsigned pipeline : 1;
    unsigned chunked : 1;
    unsigned header_only : 1; // 标记：输出不需要 body。
    unsigned expect_trailers : 1;
    unsigned keepalive : 1; // 标记：使用长连接。通过 http 版本和 Connection 标头推断
    unsigned lingering_close : 1;
    unsigned discard_body : 1;
    unsigned reading_body : 1;
    unsigned internal : 1;
    unsigned error_page : 1;
    unsigned filter_finalize : 1;
    unsigned post_action : 1;
    unsigned request_complete : 1;
    unsigned request_output : 1;
    unsigned header_sent : 1; // 标记：请求已经发送输出头
    unsigned response_sent : 1;
    unsigned expect_tested : 1;
    unsigned root_tested : 1;
    unsigned done : 1;
    unsigned logged : 1;
    unsigned terminated : 1;

    unsigned buffered : 4;

    unsigned main_filter_need_in_memory : 1;
    unsigned filter_need_in_memory : 1;
    unsigned filter_need_temporary : 1;
    unsigned preserve_body : 1;
    unsigned allow_ranges : 1;
    unsigned subrequest_ranges : 1;
    unsigned single_range : 1;
    unsigned disable_not_modified : 1;
    unsigned stat_reading : 1;
    unsigned stat_writing : 1;
    unsigned stat_processing : 1;

    unsigned background : 1;
    unsigned health_check : 1;

    /* used to parse HTTP headers */

    ngx_uint_t state;

    ngx_uint_t header_hash;
    ngx_uint_t lowcase_index;
    u_char lowcase_header[NGX_HTTP_LC_HEADER_LEN];

    u_char *header_name_start;
    u_char *header_name_end;
    u_char *header_start;
    u_char *header_end;

    /*
     * a memory that can be reused after parsing a request line
     * via ngx_http_ephemeral_t
     */

    u_char *uri_start;
    u_char *uri_end;
    u_char *uri_ext;
    u_char *args_start;
    u_char *request_start;
    u_char *request_end;
    u_char *method_end;
    u_char *schema_start;
    u_char *schema_end;
    u_char *host_start;
    u_char *host_end;

    // 协议版本
    unsigned http_minor : 16;
    unsigned http_major : 16;
};
```