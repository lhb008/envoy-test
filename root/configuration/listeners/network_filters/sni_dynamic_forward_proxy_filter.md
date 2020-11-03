SNI dynamic forward proxy {#config_network_filters_sni_dynamic_forward_proxy}
=========================

::: {.attention}
::: {.title}
Attention
:::

SNI dynamic forward proxy support should be considered alpha and not
production ready.
:::

Through the combination of
`TLS inspector <config_listener_filters_tls_inspector>`{.interpreted-text
role="ref"} listener filter, this network filter and the
`dynamic forward proxy cluster <envoy_api_msg_config.cluster.dynamic_forward_proxy.v2alpha.ClusterConfig>`{.interpreted-text
role="ref"}, Envoy supports SNI based dynamic forward proxy. The
implementation works just like the
`HTTP dynamic forward proxy <arch_overview_http_dynamic_forward_proxy>`{.interpreted-text
role="ref"}, but using the value in SNI as target host instead.

The following is a complete configuration that configures both this
filter as well as the `dynamic forward proxy cluster
<envoy_api_msg_config.cluster.dynamic_forward_proxy.v2alpha.ClusterConfig>`{.interpreted-text
role="ref"}. Both filter and cluster must be configured together and
point to the same DNS cache parameters for Envoy to operate as an SNI
dynamic forward proxy.

::: {.note}
::: {.title}
Note
:::

The following config doesn\'t terminate TLS in listener, so there is no
need to configure TLS context in cluster. The TLS handshake is passed
through by Envoy.
:::

::: {.literalinclude language="yaml"}
\_include/sni-dynamic-forward-proxy-filter.yaml
:::
