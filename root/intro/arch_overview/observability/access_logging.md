Access logging {#arch_overview_access_logs}
==============

The
`HTTP connection manager <arch_overview_http_conn_man>`{.interpreted-text
role="ref"} and `tcp proxy <arch_overview_tcp_proxy>`{.interpreted-text
role="ref"} support extensible access logging with the following
features:

-   Any number of access logs per a connection stream.
-   Customizable access log filters that allow different types of
    requests and responses to be written to different access logs.

Downstream connection access logging can be enabled using
`listener access
logs<envoy_v3_api_field_config.listener.v3.Listener.access_log>`{.interpreted-text
role="ref"}. The listener access logs complement HTTP request access
logging and can be enabled separately and independently from filter
access logs.

Access log filters {#arch_overview_access_log_filters}
------------------

Envoy supports several built-in
`access log filters<envoy_v3_api_msg_config.accesslog.v3.AccessLogFilter>`{.interpreted-text
role="ref"} and
`extension filters<envoy_v3_api_field_config.accesslog.v3.AccessLogFilter.extension_filter>`{.interpreted-text
role="ref"} that are registered at runtime.

Access logging sinks {#arch_overview_access_logs_sinks}
--------------------

Envoy supports pluggable access logging sinks. The currently supported
sinks are:

### File

-   Asynchronous IO flushing architecture. Access logging will never
    block the main network processing threads.
-   Customizable access log formats using predefined fields as well as
    arbitrary HTTP request and response headers.

### gRPC

-   Envoy can send access log messages to a gRPC access logging service.

Further reading
---------------

-   Access log `configuration <config_access_log>`{.interpreted-text
    role="ref"}.
-   File
    `access log sink <envoy_v3_api_msg_extensions.access_loggers.file.v3.FileAccessLog>`{.interpreted-text
    role="ref"}.
-   gRPC
    `Access Log Service (ALS) <envoy_v3_api_msg_extensions.access_loggers.grpc.v3.HttpGrpcAccessLogConfig>`{.interpreted-text
    role="ref"} sink.
