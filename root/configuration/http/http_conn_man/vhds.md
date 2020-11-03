Virtual Host Discovery Service (VHDS) {#config_http_conn_man_vhds}
=====================================

The virtual host discovery service (VHDS) API is an optional API that
Envoy will call to dynamically fetch
`virtual hosts <envoy_v3_api_msg_config.route.v3.VirtualHost>`{.interpreted-text
role="ref"}. A virtual host includes a name and set of domains that get
routed to it based on the incoming request\'s host header.

By default in RDS, all routes for a cluster are sent to every Envoy
instance in the mesh. This causes scaling issues as the size of the
cluster grows. The majority of this complexity can be found in the
virtual host configurations, of which most are not needed by any
individual proxy.

In order to fix this issue, the Virtual Host Discovery Service (VHDS)
protocol uses the delta xDS protocol to allow a route configuration to
be subscribed to and the necessary virtual hosts to be requested as
needed. Instead of sending all virtual hosts with a route config, using
VHDS will allow an Envoy instance to subscribe and unsubscribe from a
list of virtual hosts stored internally in the xDS management server.
The xDS management server will monitor this list and use it to filter
the configuration sent to an individual Envoy instance to only contain
the subscribed virtual hosts.

Virtual Host Resource Naming Convention
---------------------------------------

Virtual hosts in VHDS are identified by a combination of the name of the
route configuration to which the virtual host belongs as well as the
HTTP *host* header (*:authority* for HTTP2) entry. Resources should be
named as follows:

    <route configuration name>/<host entry>

Note that matching should be done from right to left since a host entry
cannot contain slashes while a route configuration name can.

Subscribing to Resources
------------------------

VHDS allows resources to be
`subscribed <xds_protocol_delta_subscribe>`{.interpreted-text
role="ref"} to using a
`DeltaDiscoveryRequest <envoy_v3_api_msg_service.discovery.v3.DeltaDiscoveryRequest>`{.interpreted-text
role="ref"} with the
`type_url <envoy_v3_api_field_service.discovery.v3.DeltaDiscoveryRequest.type_url>`{.interpreted-text
role="ref"} set to
[type.googleapis.com/envoy.config.route.v3.VirtualHost]{.title-ref} and
`resource_names_subscribe <envoy_v3_api_field_service.discovery.v3.DeltaDiscoveryRequest.resource_names_subscribe>`{.interpreted-text
role="ref"} set to a list of virtual host resource names for which it
would like configuration.

If a route for the contents of a host/authority header cannot be
resolved, the active stream is paused while a
`DeltaDiscoveryRequest <envoy_v3_api_msg_service.discovery.v3.DeltaDiscoveryRequest>`{.interpreted-text
role="ref"} is sent. When a
`DeltaDiscoveryResponse <envoy_v3_api_msg_service.discovery.v3.DeltaDiscoveryResponse>`{.interpreted-text
role="ref"} is received where one of the
`aliases <envoy_v3_api_field_service.discovery.v3.Resource.aliases>`{.interpreted-text
role="ref"} or the
`name <envoy_v3_api_field_service.discovery.v3.Resource.name>`{.interpreted-text
role="ref"} in the response exactly matches the
`resource_names_subscribe <envoy_v3_api_field_service.discovery.v3.DeltaDiscoveryRequest.resource_names_subscribe>`{.interpreted-text
role="ref"} entry from the
`DeltaDiscoveryRequest <envoy_v3_api_msg_service.discovery.v3.DeltaDiscoveryRequest>`{.interpreted-text
role="ref"}, the route configuration is updated, the stream is resumed,
and processing of the filter chain continues.

Updates to virtual hosts occur in two ways. If a virtual host was
originally sent over RDS, then the virtual host should be updated over
RDS. If a virtual host was subscribed to over VHDS, then updates will
take place over VHDS.

When a route configuration entry is updated, if the
`vhds field <envoy_v3_api_field_config.route.v3.RouteConfiguration.vhds>`{.interpreted-text
role="ref"} has changed, the virtual host table for that route
configuration is cleared, which will require that all virtual hosts be
sent again.

### Compatibility with Scoped RDS

VHDS shouldn\'t present any compatibility issues with
`scoped RDS <envoy_v3_api_msg_config.route.v3.ScopedRouteConfiguration>`{.interpreted-text
role="ref"}. Route configuration names can still be used for virtual
host matching, but with scoped RDS configured it would point to a scoped
route configuration.

However, it is important to note that using on-demand
`scoped RDS <envoy_v3_api_msg_config.route.v3.ScopedRouteConfiguration>`{.interpreted-text
role="ref"} and VHDS together will require two on-demand subscriptions
per routing scope.

-   `v2 API reference <v2_grpc_streaming_endpoints>`{.interpreted-text
    role="ref"}

### Statistics

VHDS has a statistics tree rooted at
*http.\<stat_prefix\>.vhds.\<virtual_host_name\>.*. Any `:` character in
the `virtual_host_name` name gets replaced with `_` in the stats tree.
The stats tree contains the following statistics:

  -----------------------------------------------------------------------
  Name              Type              Description
  ----------------- ----------------- -----------------------------------
  config_reload     Counter           Total API fetches that resulted in
                                      a config reload due to a different
                                      config

  empty_update      Counter           Total count of empty updates
                                      received
  -----------------------------------------------------------------------
