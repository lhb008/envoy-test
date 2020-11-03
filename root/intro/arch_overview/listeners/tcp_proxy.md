TCP proxy {#arch_overview_tcp_proxy}
=========

Since Envoy is fundamentally written as a L3/L4 server, basic L3/L4
proxy is easily implemented. The TCP proxy filter performs basic 1:1
network connection proxy between downstream clients and upstream
clusters. It can be used by itself as an stunnel replacement, or in
conjunction with other filters such as the
`MongoDB filter <arch_overview_mongo>`{.interpreted-text role="ref"} or
the `rate limit
<config_network_filters_rate_limit>`{.interpreted-text role="ref"}
filter.

The TCP proxy filter will respect the
`connection limits <envoy_v3_api_field_config.cluster.v3.CircuitBreakers.Thresholds.max_connections>`{.interpreted-text
role="ref"} imposed by each upstream cluster\'s global resource manager.
The TCP proxy filter checks with the upstream cluster\'s resource
manager if it can create a connection without going over that cluster\'s
maximum number of connections, if it can\'t the TCP proxy will not make
the connection.

TCP proxy filter
`configuration reference <config_network_filters_tcp_proxy>`{.interpreted-text
role="ref"}.
