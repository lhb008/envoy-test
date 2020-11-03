gRPC-Web {#config_http_filters_grpc_web}
========

-   gRPC `architecture overview <arch_overview_grpc>`{.interpreted-text
    role="ref"}
-   `v3 API reference <envoy_v3_api_msg_extensions.filters.http.grpc_web.v3.GrpcWeb>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.grpc_web*.

This is a filter which enables the bridging of a gRPC-Web client to a
compliant gRPC server by following
<https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md>.
