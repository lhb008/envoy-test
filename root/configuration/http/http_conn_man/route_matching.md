Route matching {#config_http_conn_man_route_table_route_matching}
==============

When Envoy matches a route, it uses the following procedure:

1.  The HTTP request\'s *host* or *:authority* header is matched to a
    `virtual host
    <envoy_v3_api_msg_config.route.v3.VirtualHost>`{.interpreted-text
    role="ref"}.
2.  Each
    `route entry <envoy_v3_api_msg_config.route.v3.Route>`{.interpreted-text
    role="ref"} in the virtual host is checked, *in order*. If there is
    a match, the route is used and no further route checks are made.
3.  Independently, each
    `virtual cluster <envoy_v3_api_msg_config.route.v3.VirtualCluster>`{.interpreted-text
    role="ref"} in the virtual host is checked, *in order*. If there is
    a match, the virtual cluster is used and no further virtual cluster
    checks are made.
