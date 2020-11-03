Route discovery service (RDS) {#config_http_conn_man_rds}
=============================

The route discovery service (RDS) API is an optional API that Envoy will
call to dynamically fetch
`route configurations <envoy_v3_api_msg_config.route.v3.RouteConfiguration>`{.interpreted-text
role="ref"}. A route configuration includes both HTTP header
modifications, virtual hosts, and the individual route entries contained
within each virtual host. Each
`HTTP connection manager filter <config_http_conn_man>`{.interpreted-text
role="ref"} can independently fetch its own route configuration via the
API. Optionally, the
`virtual host discovery service <config_http_conn_man_vhds>`{.interpreted-text
role="ref"} can be used to fetch virtual hosts separately from the route
configuration.

-   `v2 API reference <v2_grpc_streaming_endpoints>`{.interpreted-text
    role="ref"}

Statistics
----------

RDS has a `statistics <subscription_statistics>`{.interpreted-text
role="ref"} tree rooted at
*http.\<stat_prefix\>.rds.\<route_config_name\>.*. Any `:` character in
the `route_config_name` name gets replaced with `_` in the stats tree.
