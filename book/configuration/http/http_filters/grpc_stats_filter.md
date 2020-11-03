gRPC Statistics {#config_http_filters_grpc_stats}
===============

-   gRPC `architecture overview <arch_overview_grpc>`{.interpreted-text
    role="ref"}
-   `v3 API reference <envoy_v3_api_msg_extensions.filters.http.grpc_stats.v3.FilterConfig>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.grpc_stats*.
-   This filter can be enabled to emit a `filter state object
    <envoy_v3_api_msg_extensions.filters.http.grpc_stats.v3.FilterObject>`{.interpreted-text
    role="ref"}

This is a filter which enables telemetry of gRPC calls. Additionally,
the filter detects message boundaries in streaming gRPC calls and emits
the message counts for both the request and the response.

More info: wire format in [gRPC over
HTTP/2](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md).

The filter emits statistics in the *cluster.\<route target
cluster\>.grpc.* namespace. Depending on the configuration, the stats
may be prefixed with [\<grpc service\>.\<grpc method\>.]{.title-ref};
the stats in the table below are shown in this form. See the
documentation for
`individual_method_stats_allowlist <envoy_v3_api_field_extensions.filters.http.grpc_stats.v3.FilterConfig.individual_method_stats_allowlist>`{.interpreted-text
role="ref"} and
`stats_for_all_methods <envoy_v3_api_field_extensions.filters.http.grpc_stats.v3.FilterConfig.stats_for_all_methods>`{.interpreted-text
role="ref"}.

To enable *upstream_rq_time* (v3 API only) see
`enable_upstream_stats <envoy_v3_api_field_extensions.filters.http.grpc_stats.v3.FilterConfig.enable_upstream_stats>`{.interpreted-text
role="ref"}.

  ---------------------------------------------------------------------------------------
  Name                              Type              Description
  --------------------------------- ----------------- -----------------------------------
  \<grpc service\>.\<grpc           Counter           Total successful service/method
  method\>.success                                    calls

  \<grpc service\>.\<grpc           Counter           Total failed service/method calls
  method\>.failure                                    

  \<grpc service\>.\<grpc           Counter           Total service/method calls
  method\>.total                                      

  \<grpc service\>.\<grpc           Counter           Total request message count for
  method\>.request_message_count                      service/method calls

  \<grpc service\>.\<grpc           Counter           Total response message count for
  method\>.response_message_count                     service/method calls

  \<grpc service\>.\<grpc           Histogram         Request time milliseconds
  method\>.upstream_rq_time                           
  ---------------------------------------------------------------------------------------
