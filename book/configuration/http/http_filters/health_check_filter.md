Health check {#config_http_filters_health_check}
============

-   Health check filter
    `architecture overview <arch_overview_health_checking_filter>`{.interpreted-text
    role="ref"}
-   `v3 API reference <envoy_v3_api_msg_extensions.filters.http.health_check.v3.HealthCheck>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.health_check*.

::: {.note}
::: {.title}
Note
:::

Note that the filter will automatically fail health checks and set the
`x-envoy-immediate-health-check-fail
<config_http_filters_router_x-envoy-immediate-health-check-fail>`{.interpreted-text
role="ref"} header if the
`/healthcheck/fail <operations_admin_interface_healthcheck_fail>`{.interpreted-text
role="ref"} admin endpoint has been called. (The
`/healthcheck/ok <operations_admin_interface_healthcheck_ok>`{.interpreted-text
role="ref"} admin endpoint reverses this behavior).
:::
