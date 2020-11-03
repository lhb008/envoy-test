Original Destination {#config_listener_filters_original_dst}
====================

Original destination listener filter reads the SO_ORIGINAL_DST socket
option set when a connection has been redirected by an iptables REDIRECT
target, or by an iptables TPROXY target in combination with setting the
listener\'s
`transparent <envoy_v3_api_field_config.listener.v3.Listener.transparent>`{.interpreted-text
role="ref"} option. Later processing in Envoy sees the restored
destination address as the connection\'s local address, rather than the
address at which the listener is listening at. Furthermore, `an original
destination cluster <arch_overview_service_discovery_types_original_destination>`{.interpreted-text
role="ref"} may be used to forward HTTP requests or TCP connections to
the restored destination address.

-   `v2 API reference <envoy_v3_api_field_config.listener.v3.ListenerFilter.name>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.listener.original_dst*.
