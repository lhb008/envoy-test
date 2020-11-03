Rate limit {#config_thrift_filters_rate_limit}
==========

-   Global rate limiting
    `architecture overview <arch_overview_global_rate_limit>`{.interpreted-text
    role="ref"}
-   `v3 API reference <envoy_v3_api_msg_extensions.filters.network.thrift_proxy.filters.ratelimit.v3.RateLimit>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.thrift.rate_limit*.

The Thrift rate limit filter will call the rate limit service when the
request\'s route has one or more `rate limit configurations
<envoy_v3_api_field_extensions.filters.network.thrift_proxy.v3.RouteAction.rate_limits>`{.interpreted-text
role="ref"} that match the filter\'s stage setting. More than one
configuration can apply to a request. Each configuration results in a
descriptor being sent to the rate limit service.

If the rate limit service is called, and the response for any of the
descriptors is over limit, an application exception indicating an
internal error is returned.

If there is an error in calling the rate limit service or it returns an
error and `failure_mode_deny
<envoy_v3_api_field_extensions.filters.network.thrift_proxy.filters.ratelimit.v3.RateLimit.failure_mode_deny>`{.interpreted-text
role="ref"} is set to true, an application exception indicating an
internal error is returned.

Statistics {#config_thrift_filters_rate_limit_stats}
----------

The filter outputs statistics in the *cluster.\<route target
cluster\>.ratelimit.* namespace.

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                   Type              Description
  ---------------------- ----------------- ----------------------------------------------------------------------------------------------------------------------------------
  ok                     Counter           Total under limit responses from the rate limit service.

  error                  Counter           Total errors contacting the rate limit service.

  over_limit             Counter           Total over limit responses from the rate limit service.

  failure_mode_allowed   Counter           Total requests that were error(s) but were allowed through because of `failure_mode_deny
                                           <envoy_v3_api_field_extensions.filters.network.thrift_proxy.filters.ratelimit.v3.RateLimit.failure_mode_deny>`{.interpreted-text
                                           role="ref"} set to false.
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
