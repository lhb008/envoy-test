Rate limit {#config_http_filters_rate_limit}
==========

-   Global rate limiting
    `architecture overview <arch_overview_global_rate_limit>`{.interpreted-text
    role="ref"}
-   `v3 API reference <envoy_v3_api_msg_extensions.filters.http.ratelimit.v3.RateLimit>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.ratelimit*.

The HTTP rate limit filter will call the rate limit service when the
request\'s route or virtual host has one or more
`rate limit configurations<envoy_v3_api_field_config.route.v3.VirtualHost.rate_limits>`{.interpreted-text
role="ref"} that match the filter stage setting. The
`route<envoy_v3_api_field_config.route.v3.RouteAction.include_vh_rate_limits>`{.interpreted-text
role="ref"} can optionally include the virtual host rate limit
configurations. More than one configuration can apply to a request. Each
configuration results in a descriptor being sent to the rate limit
service.

If the rate limit service is called, and the response for any of the
descriptors is over limit, a 429 response is returned. The rate limit
filter also sets the
`x-envoy-ratelimited<config_http_filters_router_x-envoy-ratelimited>`{.interpreted-text
role="ref"} header.

If there is an error in calling rate limit service or rate limit service
returns an error and
`failure_mode_deny <envoy_v3_api_field_extensions.filters.http.ratelimit.v3.RateLimit.failure_mode_deny>`{.interpreted-text
role="ref"} is set to true, a 500 response is returned.

Composing Actions {#config_http_filters_rate_limit_composing_actions}
-----------------

Each
`rate limit action <envoy_v3_api_msg_config.route.v3.RateLimit>`{.interpreted-text
role="ref"} on the route or virtual host populates a descriptor entry. A
vector of descriptor entries compose a descriptor. To create more
complex rate limit descriptors, actions can be composed in any order.
The descriptor will be populated in the order the actions are specified
in the configuration.

### Example 1

For example, to generate the following descriptor:

``` {.cpp}
("generic_key", "some_value")
("source_cluster", "from_cluster")
```

The configuration would be:

``` {.yaml}
actions:
    - {source_cluster: {}}
    - {generic_key: {descriptor_value: some_value}}
```

### Example 2

If an action doesn\'t append a descriptor entry, no descriptor is
generated for the configuration.

For the following configuration:

``` {.yaml}
actions:
    - {source_cluster: {}}
    - {remote_address: {}}
    - {generic_key: {descriptor_value: some_value}}
```

If a request did not set
`x-forwarded-for<config_http_conn_man_headers_x-forwarded-for>`{.interpreted-text
role="ref"}, no descriptor is generated.

If a request sets
`x-forwarded-for<config_http_conn_man_headers_x-forwarded-for>`{.interpreted-text
role="ref"}, the the following descriptor is generated:

``` {.cpp}
("generic_key", "some_value")
("remote_address", "<trusted address from x-forwarded-for>")
("source_cluster", "from_cluster")
```

Rate Limit Override {#config_http_filters_rate_limit_rate_limit_override}
-------------------

A
`rate limit action <envoy_v3_api_msg_config.route.v3.RateLimit>`{.interpreted-text
role="ref"} can optionally contain a
`limit override <envoy_v3_api_msg_config.route.v3.RateLimit.Override>`{.interpreted-text
role="ref"}. The limit value will be appended to the descriptor produced
by the action and sent to the ratelimit service, overriding the static
service configuration.

The override can be configured to be taken from the `Dynamic Metadata
<envoy_v3_api_msg_config.core.v3.Metadata>`{.interpreted-text
role="ref"} under a specified :ref: [key
\<envoy_v3_api_msg_config.type.metadata.v3.MetadataKey\>]{.title-ref}.
If the value is misconfigured or key does not exist, the override
configuration is ignored.

### Example 3

The following configuration

``` {.yaml}
actions:
    - {generic_key: {descriptor_value: some_value}}
limit:
   metadata_key:
       key: test.filter.key
       path:
           - key: test
```

::: {#config_http_filters_rate_limit_override_dynamic_metadata}
Will lookup the value of the dynamic metadata. The value must be a
structure with integer field \"requests_per_unit\" and a string field
\"unit\" which is parseable to `RateLimitUnit enum
<envoy_v3_api_enum_type.v3.RateLimitUnit>`{.interpreted-text
role="ref"}. For example, with the following dynamic metadata the rate
limit override of 42 requests per hour will be appended to the rate
limit descriptor.
:::

``` {.yaml}
test.filter.key:
    test:
        requests_per_unit: 42
        unit: HOUR
```

Statistics
----------

The rate limit filter outputs statistics in the *cluster.\<route target
cluster\>.ratelimit.* namespace. 429 responses are emitted to the normal
cluster `dynamic HTTP statistics
<config_cluster_manager_cluster_stats_dynamic_http>`{.interpreted-text
role="ref"}.

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                   Type              Description
  ---------------------- ----------------- -----------------------------------------------------------------------------------------------------------------------------
  ok                     Counter           Total under limit responses from the rate limit service

  error                  Counter           Total errors contacting the rate limit service

  over_limit             Counter           total over limit responses from the rate limit service

  failure_mode_allowed   Counter           Total requests that were error(s) but were allowed through because of
                                           `failure_mode_deny <envoy_v3_api_field_extensions.filters.http.ratelimit.v3.RateLimit.failure_mode_deny>`{.interpreted-text
                                           role="ref"} set to false.
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------

Runtime
-------

The HTTP rate limit filter supports the following runtime settings:

ratelimit.http_filter_enabled

:   \% of requests that will call the rate limit service. Defaults
    to 100.

ratelimit.http_filter_enforcing

:   \% of requests that that will have the rate limit service decision
    enforced. Defaults to 100. This can be used to test what would
    happen before fully enforcing the outcome.

ratelimit.\<route_key\>.http_filter_enabled

:   \% of requests that will call the rate limit service for a given
    *route_key* specified in the
    `rate limit configuration <envoy_v3_api_msg_config.route.v3.RateLimit>`{.interpreted-text
    role="ref"}. Defaults to 100.
