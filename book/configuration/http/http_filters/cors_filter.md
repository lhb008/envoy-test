CORS {#config_http_filters_cors}
====

This is a filter which handles Cross-Origin Resource Sharing requests
based on route or virtual host settings. For the meaning of the headers
please refer to the pages below.

-   <https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS>
-   <https://www.w3.org/TR/cors/>
-   `v2 API reference <envoy_v3_api_msg_config.route.v3.CorsPolicy>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.cors*.

Runtime {#cors-runtime}
-------

The fraction of requests for which the filter is enabled can be
configured via the `runtime_key
<envoy_v3_api_field_config.core.v3.RuntimeFractionalPercent.runtime_key>`{.interpreted-text
role="ref"} value of the `filter_enabled
<envoy_v3_api_field_config.route.v3.CorsPolicy.filter_enabled>`{.interpreted-text
role="ref"} field.

The fraction of requests for which the filter is enabled in shadow-only
mode can be configured via the
`runtime_key <envoy_v3_api_field_config.core.v3.RuntimeFractionalPercent.runtime_key>`{.interpreted-text
role="ref"} value of the
`shadow_enabled <envoy_v3_api_field_config.route.v3.CorsPolicy.shadow_enabled>`{.interpreted-text
role="ref"} field. When enabled in shadow-only mode, the filter will
evaluate the request\'s *Origin* to determine if it\'s valid but will
not enforce any policies.

::: {.note}
::: {.title}
Note
:::

If both `filter_enabled` and `shadow_enabled` are on, the
`filter_enabled` flag will take precedence.
:::

Statistics {#cors-statistics}
----------

The CORS filter outputs statistics in the \<stat_prefix\>.cors.\*
namespace.

::: {.note}
::: {.title}
Note
:::

Requests that do not have an Origin header will be omitted from
statistics.
:::

  -----------------------------------------------------------------------
  Name              Type              Description
  ----------------- ----------------- -----------------------------------
  origin_valid      Counter           Number of requests that have a
                                      valid Origin header.

  origin_invalid    Counter           Number of requests that have an
                                      invalid Origin header.
  -----------------------------------------------------------------------
