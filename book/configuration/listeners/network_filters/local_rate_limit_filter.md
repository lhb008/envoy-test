Local rate limit {#config_network_filters_local_rate_limit}
================

-   Local rate limiting
    `architecture overview <arch_overview_local_rate_limit>`{.interpreted-text
    role="ref"}
-   `v3 API reference
    <envoy_v3_api_msg_extensions.filters.network.local_ratelimit.v3.LocalRateLimit>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.network.local_ratelimit*.

::: {.note}
::: {.title}
Note
:::

Global rate limiting is also supported via the `global rate limit filter
<config_network_filters_rate_limit>`{.interpreted-text role="ref"}.
:::

Overview
--------

The local rate limit filter applies a `token bucket
<envoy_v3_api_field_extensions.filters.network.local_ratelimit.v3.LocalRateLimit.token_bucket>`{.interpreted-text
role="ref"} rate limit to incoming connections that are processed by the
filter\'s filter chain. Each connection processed by the filter utilizes
a single token, and if no tokens are available, the connection will be
immediately closed without further filter iteration.

::: {.note}
::: {.title}
Note
:::

In the current implementation each filter and filter chain has an
independent rate limit.
:::

Statistics {#config_network_filters_local_rate_limit_stats}
----------

Every configured local rate limit filter has statistics rooted at
*local_ratelimit.\<stat_prefix\>.* with the following statistics:

  -----------------------------------------------------------------------
  Name              Type              Description
  ----------------- ----------------- -----------------------------------
  rate_limited      Counter           Total connections that have been
                                      closed due to rate limit exceeded

  -----------------------------------------------------------------------

Runtime
-------

The local rate limit filter can be runtime feature flagged via the
`enabled
<envoy_v3_api_field_extensions.filters.network.local_ratelimit.v3.LocalRateLimit.runtime_enabled>`{.interpreted-text
role="ref"} configuration field.
