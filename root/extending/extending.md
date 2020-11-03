Extending Envoy for custom use cases {#extending}
====================================

The Envoy architecture makes it fairly easily extensible via a variety
of different extension types including:

-   `Access loggers <arch_overview_access_logs>`{.interpreted-text
    role="ref"}
-   `Access log filters <arch_overview_access_log_filters>`{.interpreted-text
    role="ref"}
-   `Clusters <arch_overview_service_discovery>`{.interpreted-text
    role="ref"}
-   `Listener filters <arch_overview_listener_filters>`{.interpreted-text
    role="ref"}
-   `Network filters <arch_overview_network_filters>`{.interpreted-text
    role="ref"}
-   `HTTP filters <arch_overview_http_filters>`{.interpreted-text
    role="ref"}
-   `gRPC credential providers <arch_overview_grpc>`{.interpreted-text
    role="ref"}
-   `Health checkers <arch_overview_health_checking>`{.interpreted-text
    role="ref"}
-   `Resource monitors <arch_overview_overload_manager>`{.interpreted-text
    role="ref"}
-   `Retry implementations <arch_overview_http_routing_retry>`{.interpreted-text
    role="ref"}
-   `Stat sinks <arch_overview_statistics>`{.interpreted-text
    role="ref"}
-   `Tracers <arch_overview_tracing>`{.interpreted-text role="ref"}
-   `Request ID <arch_overview_tracing>`{.interpreted-text role="ref"}
-   `Transport sockets <envoy_v3_api_field_extensions.transport_sockets.tls.v3.CommonTlsContext.CertificateProvider.typed_config>`{.interpreted-text
    role="ref"}
-   BoringSSL private key methods
-   `Watchdog action <envoy_v3_api_msg_config.bootstrap.v3.Watchdog.WatchdogAction>`{.interpreted-text
    role="ref"}
-   `Internal redirect policy <envoy_v3_api_field_config.route.v3.InternalRedirectPolicy.predicates>`{.interpreted-text
    role="ref"}
-   `Compression libraries <arch_overview_compression_libraries>`{.interpreted-text
    role="ref"}

As of this writing there is no high level extension developer
documentation. The
`existing extensions <source/extensions>`{.interpreted-text role="repo"}
are a good way to learn what is possible.

An example of how to add a network filter and structure the repository
and build dependencies can be found at
[envoy-filter-example](https://github.com/envoyproxy/envoy-filter-example).
