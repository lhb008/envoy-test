HTTP header sanitizing {#config_http_conn_man_header_sanitizing}
======================

For security reasons, Envoy will \"sanitize\" various incoming HTTP
headers depending on whether the request is an internal or external
request. The sanitizing action depends on the header and may result in
addition, removal, or modification. Ultimately, whether the request is
considered internal or external is governed by the
`x-forwarded-for <config_http_conn_man_headers_x-forwarded-for>`{.interpreted-text
role="ref"} header (please read the linked section carefully as how
Envoy populates the header is complex and depends on the
`use_remote_address
<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.use_remote_address>`{.interpreted-text
role="ref"} setting). In addition, the `internal_address_config
<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.internal_address_config>`{.interpreted-text
role="ref"} setting can be used to configure the internal/external
determination.

Envoy will potentially sanitize the following headers:

-   `x-envoy-decorator-operation <config_http_filters_router_x-envoy-decorator-operation>`{.interpreted-text
    role="ref"}
-   `x-envoy-downstream-service-cluster
    <config_http_conn_man_headers_downstream-service-cluster>`{.interpreted-text
    role="ref"}
-   `x-envoy-downstream-service-node <config_http_conn_man_headers_downstream-service-node>`{.interpreted-text
    role="ref"}
-   `x-envoy-expected-rq-timeout-ms <config_http_filters_router_x-envoy-expected-rq-timeout-ms>`{.interpreted-text
    role="ref"}
-   `x-envoy-external-address <config_http_conn_man_headers_x-envoy-external-address>`{.interpreted-text
    role="ref"}
-   `x-envoy-force-trace <config_http_conn_man_headers_x-envoy-force-trace>`{.interpreted-text
    role="ref"}
-   `x-envoy-internal <config_http_conn_man_headers_x-envoy-internal>`{.interpreted-text
    role="ref"}
-   `x-envoy-ip-tags <config_http_filters_ip_tagging>`{.interpreted-text
    role="ref"}
-   `x-envoy-max-retries <config_http_filters_router_x-envoy-max-retries>`{.interpreted-text
    role="ref"}
-   `x-envoy-retry-grpc-on <config_http_filters_router_x-envoy-retry-grpc-on>`{.interpreted-text
    role="ref"}
-   `x-envoy-retry-on <config_http_filters_router_x-envoy-retry-on>`{.interpreted-text
    role="ref"}
-   `x-envoy-upstream-alt-stat-name <config_http_filters_router_x-envoy-upstream-alt-stat-name>`{.interpreted-text
    role="ref"}
-   `x-envoy-upstream-rq-per-try-timeout-ms
    <config_http_filters_router_x-envoy-upstream-rq-per-try-timeout-ms>`{.interpreted-text
    role="ref"}
-   `x-envoy-upstream-rq-timeout-alt-response
    <config_http_filters_router_x-envoy-upstream-rq-timeout-alt-response>`{.interpreted-text
    role="ref"}
-   `x-envoy-upstream-rq-timeout-ms <config_http_filters_router_x-envoy-upstream-rq-timeout-ms>`{.interpreted-text
    role="ref"}
-   `x-forwarded-client-cert <config_http_conn_man_headers_x-forwarded-client-cert>`{.interpreted-text
    role="ref"}
-   `x-forwarded-for <config_http_conn_man_headers_x-forwarded-for>`{.interpreted-text
    role="ref"}
-   `x-forwarded-proto <config_http_conn_man_headers_x-forwarded-proto>`{.interpreted-text
    role="ref"}
-   `x-request-id <config_http_conn_man_headers_x-request-id>`{.interpreted-text
    role="ref"}
