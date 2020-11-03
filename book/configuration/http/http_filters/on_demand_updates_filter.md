On-demand VHDS and S/RDS Updates {#config_http_filters_on_demand}
================================

The on demand filter can be used to support either on demand VHDS or
S/RDS update if configured in the filter chain.

The on-demand update filter can be used to request a
`virtual host <envoy_v3_api_msg_config.route.v3.VirtualHost>`{.interpreted-text
role="ref"} data if it\'s not already present in the
`Route Configuration <envoy_v3_api_msg_config.route.v3.RouteConfiguration>`{.interpreted-text
role="ref"}. The contents of the *Host* or *:authority* header is used
to create the on-demand request. For an on-demand request to be created,
`VHDS <envoy_v3_api_field_config.route.v3.RouteConfiguration.vhds>`{.interpreted-text
role="ref"} must be enabled and either *Host* or *:authority* header be
present.

The on-demand update filter can also be used to request a *Route
Configuration* data if RouteConfiguration is specified to be loaded on
demand in the
`Scoped RouteConfiguration <envoy_v3_api_msg_config.route.v3.ScopedRouteConfiguration>`{.interpreted-text
role="ref"}. The contents of the HTTP header is used to find the scope
and create the on-demand request.

On-demand VHDS and on-demand S/RDS can not be used at the same time at
this point.

Configuration
-------------

-   `v3 API reference <envoy_v3_api_msg_extensions.filters.http.on_demand.v3.OnDemand>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.on_demand*.
-   The filter should be placed before *envoy.filters.http.router*
    filter in the HttpConnectionManager\'s filter chain.
