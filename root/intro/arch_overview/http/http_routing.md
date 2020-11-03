HTTP routing {#arch_overview_http_routing}
============

Envoy includes an HTTP
`router filter <config_http_filters_router>`{.interpreted-text
role="ref"} which can be installed to perform advanced routing tasks.
This is useful both for handling edge traffic (traditional reverse proxy
request handling) as well as for building a service to service Envoy
mesh (typically via routing on the host/authority HTTP header to reach a
particular upstream service cluster). Envoy also has the ability to be
configured as forward proxy. In the forward proxy configuration, mesh
clients can participate by appropriately configuring their http proxy to
be an Envoy. At a high level the router takes an incoming HTTP request,
matches it to an upstream cluster, acquires a
`connection pool <arch_overview_conn_pool>`{.interpreted-text
role="ref"} to a host in the upstream cluster, and forwards the request.
The router filter supports the following features:

-   Virtual hosts that map domains/authorities to a set of routing
    rules.
-   Prefix and exact path matching rules (both `case sensitive
    <envoy_v3_api_field_config.route.v3.RouteMatch.case_sensitive>`{.interpreted-text
    role="ref"} and case insensitive). Regex/slug matching is not
    currently supported, mainly because it makes it difficult/impossible
    to programmatically determine whether routing rules conflict with
    each other. For this reason we don't recommend regex/slug routing at
    the reverse proxy level, however we may add support in the future
    depending on demand.
-   `TLS redirection <envoy_v3_api_field_config.route.v3.VirtualHost.require_tls>`{.interpreted-text
    role="ref"} at the virtual host level.
-   `Path <envoy_v3_api_field_config.route.v3.RedirectAction.path_redirect>`{.interpreted-text
    role="ref"}/`host
    <envoy_v3_api_field_config.route.v3.RedirectAction.host_redirect>`{.interpreted-text
    role="ref"} redirection at the route level.
-   `Direct (non-proxied) HTTP responses <arch_overview_http_routing_direct_response>`{.interpreted-text
    role="ref"} at the route level.
-   `Explicit host rewriting <envoy_v3_api_field_extensions.filters.http.aws_request_signing.v3.AwsRequestSigning.host_rewrite>`{.interpreted-text
    role="ref"}.
-   `Automatic host rewriting <envoy_v3_api_field_config.route.v3.RouteAction.auto_host_rewrite>`{.interpreted-text
    role="ref"} based on the DNS name of the selected upstream host.
-   `Prefix rewriting <envoy_v3_api_field_config.route.v3.RedirectAction.prefix_rewrite>`{.interpreted-text
    role="ref"}.
-   `Path rewriting using a regular expression and capture groups <envoy_v3_api_field_config.route.v3.RouteAction.regex_rewrite>`{.interpreted-text
    role="ref"}.
-   `Request retries <arch_overview_http_routing_retry>`{.interpreted-text
    role="ref"} specified either via HTTP header or via route
    configuration.
-   Request timeout specified either via `HTTP
    header <config_http_filters_router_headers_consumed>`{.interpreted-text
    role="ref"} or via `route configuration
    <envoy_v3_api_field_config.route.v3.RouteAction.timeout>`{.interpreted-text
    role="ref"}.
-   `Request hedging <arch_overview_http_routing_hedging>`{.interpreted-text
    role="ref"} for retries in response to a request (per try) timeout.
-   Traffic shifting from one upstream cluster to another via
    `runtime values
    <envoy_v3_api_field_config.route.v3.RouteMatch.runtime_fraction>`{.interpreted-text
    role="ref"} (see `traffic shifting/splitting
    <config_http_conn_man_route_table_traffic_splitting>`{.interpreted-text
    role="ref"}).
-   Traffic splitting across multiple upstream clusters using
    `weight/percentage-based routing
    <envoy_v3_api_field_config.route.v3.RouteAction.weighted_clusters>`{.interpreted-text
    role="ref"} (see `traffic shifting/splitting
    <config_http_conn_man_route_table_traffic_splitting_split>`{.interpreted-text
    role="ref"}).
-   Arbitrary header matching
    `routing rules <envoy_v3_api_msg_config.route.v3.HeaderMatcher>`{.interpreted-text
    role="ref"}.
-   Virtual cluster specifications. A virtual cluster is specified at
    the virtual host level and is used by Envoy to generate additional
    statistics on top of the standard cluster level ones. Virtual
    clusters can use regex matching.
-   `Priority <arch_overview_http_routing_priority>`{.interpreted-text
    role="ref"} based routing.
-   `Hash policy <envoy_v3_api_field_config.route.v3.RouteAction.hash_policy>`{.interpreted-text
    role="ref"} based routing.
-   `Absolute urls <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.http_protocol_options>`{.interpreted-text
    role="ref"} are supported for non-tls forward proxies.

Route Scope {#arch_overview_http_routing_route_scope}
-----------

Scoped routing enables Envoy to put constraints on search space of
domains and route rules. A
`Route Scope<envoy_api_msg_ScopedRouteConfiguration>`{.interpreted-text
role="ref"} associates a key with a
`route table <arch_overview_http_routing_route_table>`{.interpreted-text
role="ref"}. For each request, a scope key is computed dynamically by
the HTTP connection manager to pick the
`route table<envoy_api_msg_RouteConfiguration>`{.interpreted-text
role="ref"}. RouteConfiguration associated with scope can be loaded on
demand with
`v3 API reference <envoy_v3_api_msg_extensions.filters.http.on_demand.v3.OnDemand>`{.interpreted-text
role="ref"} configured and on demand filed in protobuf set to true.

The Scoped RDS (SRDS) API contains a set of
`Scopes <envoy_v3_api_msg_config.route.v3.ScopedRouteConfiguration>`{.interpreted-text
role="ref"} resources, each defining independent routing configuration,
along with a
`ScopeKeyBuilder <envoy_v3_api_msg_extensions.filters.network.http_connection_manager.v3.ScopedRoutes.ScopeKeyBuilder>`{.interpreted-text
role="ref"} defining the key construction algorithm used by Envoy to
look up the scope corresponding to each request.

For example, for the following scoped route configuration, Envoy will
look into the \"addr\" header value, split the header value by \";\"
first, and use the first value for key \'x-foo-key\' as the scope key.
If the \"addr\" header value is
\"foo=1;x-foo-key=127.0.0.1;x-bar-key=1.1.1.1\", then \"127.0.0.1\" will
be computed as the scope key to look up for corresponding route
configuration.

``` {.yaml}
name: scope_by_addr
fragments:
  - header_value_extractor:
      name: Addr
      element_separator: ;
      element:
        key: x-foo-key
        separator: =
```

::: {#arch_overview_http_routing_route_table}
For a key to match a
`ScopedRouteConfiguration<envoy_v3_api_msg_config.route.v3.ScopedRouteConfiguration>`{.interpreted-text
role="ref"}, the number of fragments in the computed key has to match
that of the
`ScopedRouteConfiguration<envoy_v3_api_msg_config.route.v3.ScopedRouteConfiguration>`{.interpreted-text
role="ref"}. Then fragments are matched in order. A missing
fragment(treated as NULL) in the built key makes the request unable to
match any scope, i.e. no route entry can be found for the request.
:::

Route table
-----------

The `configuration <config_http_conn_man>`{.interpreted-text role="ref"}
for the HTTP connection manager owns the `route
table <envoy_v3_api_msg_config.route.v3.RouteConfiguration>`{.interpreted-text
role="ref"} that is used by all configured HTTP filters. Although the
router filter is the primary consumer of the route table, other filters
also have access in case they want to make decisions based on the
ultimate destination of the request. For example, the built in rate
limit filter consults the route table to determine whether the global
rate limit service should be called based on the route. The connection
manager makes sure that all calls to acquire a route are stable for a
particular request, even if the decision involves randomness (e.g. in
the case of a runtime configuration route rule).

Retry semantics {#arch_overview_http_routing_retry}
---------------

Envoy allows retries to be configured both in the `route configuration
<envoy_v3_api_field_config.route.v3.RouteAction.retry_policy>`{.interpreted-text
role="ref"} as well as for specific requests via `request
headers <config_http_filters_router_headers_consumed>`{.interpreted-text
role="ref"}. The following configurations are possible:

-   **Maximum number of retries**: Envoy will continue to retry any
    number of times. The intervals between retries are decided either by
    an exponential backoff algorithm (the default), or based on feedback
    from the upstream server via headers (if present). Additionally,
    *all retries are contained within the overall request timeout*. This
    avoids long request times due to a large number of retries.
-   **Retry conditions**: Envoy can retry on different types of
    conditions depending on application requirements. For example,
    network failure, all 5xx response codes, idempotent 4xx response
    codes, etc.
-   **Retry budgets**: Envoy can limit the proportion of active requests
    via
    `retry budgets <envoy_v3_api_field_config.cluster.v3.CircuitBreakers.Thresholds.retry_budget>`{.interpreted-text
    role="ref"} that can be retries to prevent their contribution to
    large increases in traffic volume.
-   **Host selection retry plugins**: Envoy can be configured to apply
    additional logic to the host selection logic when selecting hosts
    for retries. Specifying a
    `retry host predicate <envoy_v3_api_field_config.route.v3.RetryPolicy.retry_host_predicate>`{.interpreted-text
    role="ref"} allows for reattempting host selection when certain
    hosts are selected (e.g. when an already attempted host is
    selected), while a
    `retry priority <envoy_v3_api_field_config.route.v3.RetryPolicy.retry_priority>`{.interpreted-text
    role="ref"} can be configured to adjust the priority load used when
    selecting a priority for retries.

Note that Envoy retries requests when `x-envoy-overloaded
<config_http_filters_router_x-envoy-overloaded_set>`{.interpreted-text
role="ref"} is present. It is recommended to either configure
`retry budgets (preferred) <envoy_api_field_cluster.CircuitBreakers.Thresholds.retry_budget>`{.interpreted-text
role="ref"} or set
`maximum active retries circuit breaker <arch_overview_circuit_break>`{.interpreted-text
role="ref"} to an appropriate value to avoid retry storms.

Request Hedging {#arch_overview_http_routing_hedging}
---------------

Envoy supports request hedging which can be enabled by specifying a
`hedge
policy <envoy_v3_api_msg_config.route.v3.HedgePolicy>`{.interpreted-text
role="ref"}. This means that Envoy will race multiple simultaneous
upstream requests and return the response associated with the first
acceptable response headers to the downstream. The retry policy is used
to determine whether a response should be returned or whether more
responses should be awaited.

Currently hedging can only be performed in response to a request
timeout. This means that a retry request will be issued without
canceling the initial timed-out request and a late response will be
awaited. The first \"good\" response according to retry policy will be
returned downstream.

The implementation ensures that the same upstream request is not retried
twice. This might otherwise occur if a request times out and then
results in a 5xx response, creating two retriable events.

Priority routing {#arch_overview_http_routing_priority}
----------------

Envoy supports priority routing at the
`route <envoy_v3_api_msg_config.route.v3.Route>`{.interpreted-text
role="ref"} level. The current priority implementation uses different
`connection pool <arch_overview_conn_pool>`{.interpreted-text
role="ref"} and
`circuit breaking <config_cluster_manager_cluster_circuit_breakers>`{.interpreted-text
role="ref"} settings for each priority level. This means that even for
HTTP/2 requests, two physical connections will be used to an upstream
host. In the future Envoy will likely support true HTTP/2 priority over
a single connection.

The currently supported priorities are *default* and *high*.

Direct responses {#arch_overview_http_routing_direct_response}
----------------

Envoy supports the sending of \"direct\" responses. These are
preconfigured HTTP responses that do not require proxying to an upstream
server.

There are two ways to specify a direct response in a Route:

-   Set the
    `direct_response <envoy_v3_api_field_config.route.v3.Route.direct_response>`{.interpreted-text
    role="ref"} field. This works for all HTTP response statuses.
-   Set the
    `redirect <envoy_v3_api_field_config.route.v3.Route.redirect>`{.interpreted-text
    role="ref"} field. This works for redirect response statuses only,
    but it simplifies the setting of the *Location* header.

A direct response has an HTTP status code and an optional body. The
Route configuration can specify the response body inline or specify the
pathname of a file containing the body. If the Route configuration
specifies a file pathname, Envoy will read the file upon configuration
load and cache the contents.

::: {.attention}
::: {.title}
Attention
:::

If a response body is specified, it must be no more than 4KB in size,
regardless of whether it is provided inline or in a file. Envoy
currently holds the entirety of the body in memory, so the 4KB limit is
intended to keep the proxy\'s memory footprint from growing too large.
:::

If **response_headers_to_add** has been set for the Route or the
enclosing Virtual Host, Envoy will include the specified headers in the
direct HTTP response.
