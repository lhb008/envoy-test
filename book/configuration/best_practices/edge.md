Configuring Envoy as an edge proxy {#best_practices_edge}
==================================

Envoy is a production-ready edge proxy, however, the default settings
are tailored for the service mesh use case, and some values need to be
adjusted when using Envoy as an edge proxy.

TCP proxies should configure:

-   restrict access to the admin endpoint,
-   `overload_manager <config_overload_manager>`{.interpreted-text
    role="ref"},
-   `listener buffer limits <envoy_v3_api_field_config.listener.v3.Listener.per_connection_buffer_limit_bytes>`{.interpreted-text
    role="ref"} to 32 KiB,
-   `cluster buffer limits <envoy_v3_api_field_config.cluster.v3.Cluster.per_connection_buffer_limit_bytes>`{.interpreted-text
    role="ref"} to 32 KiB.

HTTP proxies should additionally configure:

-   `use_remote_address <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.use_remote_address>`{.interpreted-text
    role="ref"} to true (to avoid consuming HTTP headers from external
    clients, see
    `HTTP header sanitizing <config_http_conn_man_header_sanitizing>`{.interpreted-text
    role="ref"} for details),
-   `connection and stream timeouts <faq_configuration_timeouts>`{.interpreted-text
    role="ref"},
-   `HTTP/2 maximum concurrent streams limit <envoy_v3_api_field_config.core.v3.Http2ProtocolOptions.max_concurrent_streams>`{.interpreted-text
    role="ref"} to 100,
-   `HTTP/2 initial stream window size limit <envoy_v3_api_field_config.core.v3.Http2ProtocolOptions.initial_stream_window_size>`{.interpreted-text
    role="ref"} to 64 KiB,
-   `HTTP/2 initial connection window size limit <envoy_v3_api_field_config.core.v3.Http2ProtocolOptions.initial_connection_window_size>`{.interpreted-text
    role="ref"} to 1 MiB.
-   `headers_with_underscores_action setting <envoy_v3_api_field_config.core.v3.HttpProtocolOptions.headers_with_underscores_action>`{.interpreted-text
    role="ref"} to REJECT_REQUEST, to protect upstream services that
    treat \'\_\' and \'-\' as interchangeable.
-   `Listener connection limits. <config_listeners_runtime>`{.interpreted-text
    role="ref"}
-   `Global downstream connection limits <config_overload_manager>`{.interpreted-text
    role="ref"}.

The following is a YAML example of the above recommendation (taken from
the `Google VRP
<arch_overview_google_vrp>`{.interpreted-text role="ref"} edge server
configuration):

::: {.literalinclude language="yaml"}
\_include/edge.yaml
:::
