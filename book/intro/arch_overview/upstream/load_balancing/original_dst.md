Original destination {#arch_overview_load_balancing_types_original_destination}
====================

This is a special purpose load balancer that can only be used with
`an original destination
cluster <arch_overview_service_discovery_types_original_destination>`{.interpreted-text
role="ref"}. Upstream host is selected based on the downstream
connection metadata, i.e., connections are opened to the same address as
the destination address of the incoming connection was before the
connection was redirected to Envoy. New destinations are added to the
cluster by the load balancer on-demand, and the cluster
`periodically <envoy_v3_api_field_config.cluster.v3.Cluster.cleanup_interval>`{.interpreted-text
role="ref"} cleans out unused hosts from the cluster. No other
`load balancing policy <envoy_v3_api_field_config.cluster.v3.Cluster.lb_policy>`{.interpreted-text
role="ref"} can be used with original destination clusters.

Original destination host request header {#arch_overview_load_balancing_types_original_destination_request_header}
----------------------------------------

Envoy can also pick up the original destination from a HTTP header
called
`x-envoy-original-dst-host <config_http_conn_man_headers_x-envoy-original-dst-host>`{.interpreted-text
role="ref"}. Please note that fully resolved IP address should be passed
in this header. For example if a request has to be routed to a host with
IP address 10.195.16.237 at port 8888, the request header value should
be set as `10.195.16.237:8888`.
