External Authorization {#arch_overview_ext_authz}
======================

-   `Network filter configuration <config_network_filters_ext_authz>`{.interpreted-text
    role="ref"}.
-   `HTTP filter configuration <config_http_filters_ext_authz>`{.interpreted-text
    role="ref"}.

The External authorization filter calls an authorization service to
check if the incoming request is authorized or not. The filter can be
either configured as a
`network filter <config_network_filters_ext_authz>`{.interpreted-text
role="ref"}, or as a
`HTTP filter <config_http_filters_ext_authz>`{.interpreted-text
role="ref"} or both. If the request is deemed unauthorized by the
network filter then the connection will be closed. If the request is
deemed unauthorized at the HTTP filter the request will be denied with
403 (Forbidden) response.

::: {.tip}
::: {.title}
Tip
:::

It is recommended that these filters are configured as the first filter
in the filter chain so that requests are authorized prior to rest of the
filters processing the request.
:::

The external authorization service cluster may be either statically
configured or configured via the
`Cluster Discovery Service <config_cluster_manager_cds>`{.interpreted-text
role="ref"}. If the external service is not available when a request
comes in then whether the request is authorized or not is defined by the
configuration setting of *failure_mode_allow* configuration in the
applicable
`network filter <envoy_v3_api_msg_extensions.filters.network.ext_authz.v3.ExtAuthz>`{.interpreted-text
role="ref"} or
`HTTP filter <envoy_v3_api_msg_extensions.filters.http.ext_authz.v3.ExtAuthz>`{.interpreted-text
role="ref"}. If it is set to true then the request will be permitted
(fail open) otherwise it will be denied. The default setting is *false*.

Service Definition
------------------

The context of the traffic is passed on to an external authorization
service using the service definition listed here. The content of the
request that are passed to an authorization service is specified by
`CheckRequest <envoy_v3_api_msg_service.auth.v3.CheckRequest>`{.interpreted-text
role="ref"}.

::: {.toctree glob="" maxdepth="2"}
../../../api-v3/service/auth/v3/\*
:::
