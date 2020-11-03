Router {#config_thrift_filters_router}
======

The router filter implements Thrift forwarding. It will be used in
almost all Thrift proxying scenarios. The filter\'s main job is to
follow the instructions specified in the configured
`route table <envoy_v3_api_msg_extensions.filters.network.thrift_proxy.v3.RouteConfiguration>`{.interpreted-text
role="ref"}.

-   `v3 API reference <envoy_v3_api_msg_config.filter.thrift.router.v2alpha1.Router>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.thrift.router*.

Statistics
----------

The filter outputs statistics in the *thrift.\<stat_prefix\>.*
namespace.

  ------------------------------------------------------------------------------------
  Name                           Type              Description
  ------------------------------ ----------------- -----------------------------------
  route_missing                  Counter           Total requests with no route found.

  unknown_cluster                Counter           Total requests with a route that
                                                   has an unknown cluster.

  upstream_rq_maintenance_mode   Counter           Total requests with a destination
                                                   cluster in maintenance mode.

  no_healthy_upstream            Counter           Total requests with no healthy
                                                   upstream endpoints available.
  ------------------------------------------------------------------------------------
