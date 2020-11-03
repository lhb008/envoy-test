Rate limit {#config_network_filters_rate_limit}
==========

-   Global rate limiting
    `architecture overview <arch_overview_global_rate_limit>`{.interpreted-text
    role="ref"}
-   `v3 API reference <envoy_v3_api_msg_extensions.filters.network.ratelimit.v3.RateLimit>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.network.ratelimit*.

::: {.note}
::: {.title}
Note
:::

Local rate limiting is also supported via the `local rate limit filter
<config_network_filters_local_rate_limit>`{.interpreted-text
role="ref"}.
:::

Statistics {#config_network_filters_rate_limit_stats}
----------

Every configured rate limit filter has statistics rooted at
*ratelimit.\<stat_prefix\>.* with the following statistics:

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                   Type              Description
  ---------------------- ----------------- --------------------------------------------------------------------------------------------------------------------------------
  total                  Counter           Total requests to the rate limit service

  error                  Counter           Total errors contacting the rate limit service

  over_limit             Counter           Total over limit responses from the rate limit service

  ok                     Counter           Total under limit responses from the rate limit service

  cx_closed              Counter           Total connections closed due to an over limit response from the rate limit service

  active                 Gauge             Total active requests to the rate limit service

  failure_mode_allowed   Counter           Total requests that were error(s) but were allowed through because of
                                           `failure_mode_deny <envoy_v3_api_field_extensions.filters.network.ratelimit.v3.RateLimit.failure_mode_deny>`{.interpreted-text
                                           role="ref"} set to false.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Runtime
-------

The network rate limit filter supports the following runtime settings:

ratelimit.tcp_filter_enabled

:   \% of connections that will call the rate limit service. Defaults
    to 100.

ratelimit.tcp_filter_enforcing

:   \% of connections that will call the rate limit service and enforce
    the decision. Defaults to 100. This can be used to test what would
    happen before fully enforcing the outcome.
