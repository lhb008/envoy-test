Local rate limit {#config_http_filters_local_rate_limit}
================

-   Local rate limiting
    `architecture overview <arch_overview_local_rate_limit>`{.interpreted-text
    role="ref"}
-   `v3 API reference <envoy_v3_api_msg_extensions.filters.http.local_ratelimit.v3.LocalRateLimit>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.local_ratelimit*.

The HTTP local rate limit filter applies a `token bucket
<envoy_v3_api_field_extensions.filters.http.local_ratelimit.v3.LocalRateLimit.token_bucket>`{.interpreted-text
role="ref"} rate limit when the request\'s route or virtual host has a
per filter
`local rate limit configuration <envoy_v3_api_msg_extensions.filters.http.local_ratelimit.v3.LocalRateLimit>`{.interpreted-text
role="ref"}.

If the local rate limit token bucket is checked, and there are no token
availables, a 429 response is returned (the response is configurable).
The local rate limit filter also sets the
`x-envoy-ratelimited<config_http_filters_router_x-envoy-ratelimited>`{.interpreted-text
role="ref"} header. Additional response headers may be configured.

Example configuration
---------------------

Example filter configuration for a globally set rate limiter (e.g.: all
vhosts/routes share the same token bucket):

``` {.yaml}
name: envoy.filters.http.local_ratelimit
typed_config:
  "@type": type.googleapis.com/envoy.extensions.filters.http.local_ratelimit.v3.LocalRateLimit
  stat_prefix: http_local_rate_limiter
  token_bucket:
    max_tokens: 10000
    tokens_per_fill: 1000
    fill_interval: 1s
  filter_enabled:
    runtime_key: local_rate_limit_enabled
    default_value:
      numerator: 100
      denominator: HUNDRED
  filter_enforced:
    runtime_key: local_rate_limit_enforced
    default_value:
      numerator: 100
      denominator: HUNDRED
  response_headers_to_add:
    - append: false
      header:
        key: x-local-rate-limit
        value: 'true'
```

Example filter configuration for a globally disabled rate limiter but
enabled for a specific route:

``` {.yaml}
name: envoy.filters.http.local_ratelimit
typed_config:
  "@type": type.googleapis.com/envoy.extensions.filters.http.local_ratelimit.v3.LocalRateLimit
  stat_prefix: http_local_rate_limiter
```

The route specific configuration:

``` {.yaml}
route_config:
  name: local_route
  virtual_hosts:
  - name: local_service
    domains: ["*"]
    routes:
    - match: { prefix: "/path/with/rate/limit" }
      route: { cluster: service_protected_by_rate_limit }
      typed_per_filter_config:
        envoy.filters.http.local_ratelimit:
          "@type": type.googleapis.com/envoy.extensions.filters.http.local_ratelimit.v3.LocalRateLimit
          token_bucket:
            max_tokens: 10000
            tokens_per_fill: 1000
            fill_interval: 1s
          filter_enabled:
            runtime_key: local_rate_limit_enabled
            default_value:
              numerator: 100
              denominator: HUNDRED
          filter_enforced:
            runtime_key: local_rate_limit_enforced
            default_value:
              numerator: 100
              denominator: HUNDRED
          response_headers_to_add:
            - append: false
              header:
                key: x-local-rate-limit
                value: 'true'
    - match: { prefix: "/" }
      route: { cluster: default_service }
```

Note that if this filter is configured as globally disabled and there
are no virtual host or route level token buckets, no rate limiting will
be applied.

Statistics
----------

The local rate limit filter outputs statistics in the
*\<stat_prefix\>.http_local_rate_limit.* namespace. 429 responses \-- or
the configured status code \-- are emitted to the normal cluster
`dynamic HTTP statistics
<config_cluster_manager_cluster_stats_dynamic_http>`{.interpreted-text
role="ref"}.

  -----------------------------------------------------------------------
  Name              Type              Description
  ----------------- ----------------- -----------------------------------
  enabled           Counter           Total number of requests for which
                                      the rate limiter was consulted

  ok                Counter           Total under limit responses from
                                      the token bucket

  rate_limited      Counter           Total responses without an
                                      available token (but not
                                      necessarily enforced)

  enforced          Counter           Total number of requests for which
                                      rate limiting was applied (e.g.:
                                      429 returned)
  -----------------------------------------------------------------------

Runtime {#config_http_filters_local_rate_limit_runtime}
-------

The HTTP rate limit filter supports the following runtime fractional
settings:

http_filter_enabled

:   \% of requests that will check the local rate limit decision, but
    not enforce, for a given *route_key* specified in the
    `local rate limit configuration <envoy_v3_api_msg_extensions.filters.http.local_ratelimit.v3.LocalRateLimit>`{.interpreted-text
    role="ref"}. Defaults to 0.

http_filter_enforcing

:   \% of requests that will enforce the local rate limit decision for a
    given *route_key* specified in the
    `local rate limit configuration <envoy_v3_api_msg_extensions.filters.http.local_ratelimit.v3.LocalRateLimit>`{.interpreted-text
    role="ref"}. Defaults to 0. This can be used to test what would
    happen before fully enforcing the outcome.
