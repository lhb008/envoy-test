Bootstrap configuration {#config_overview_bootstrap}
=======================

To use the xDS API, it\'s necessary to supply a bootstrap configuration
file. This provides static server configuration and configures Envoy to
access `dynamic
configuration if needed <arch_overview_dynamic_config>`{.interpreted-text
role="ref"}. This is supplied on the command-line via the
`-c`{.interpreted-text role="option"} flag, i.e.:

``` {.console}
./envoy -c <path to config>.{json,yaml,pb,pb_text}
```

where the extension reflects the underlying config representation.

The
`Bootstrap <envoy_v3_api_msg_config.bootstrap.v3.Bootstrap>`{.interpreted-text
role="ref"} message is the root of the configuration. A key concept in
the
`Bootstrap <envoy_v3_api_msg_config.bootstrap.v3.Bootstrap>`{.interpreted-text
role="ref"} message is the distinction between static and dynamic
resources. Resources such as a
`Listener <envoy_v3_api_msg_config.listener.v3.Listener>`{.interpreted-text
role="ref"} or `Cluster
<envoy_v3_api_msg_config.cluster.v3.Cluster>`{.interpreted-text
role="ref"} may be supplied either statically in
`static_resources <envoy_v3_api_field_config.bootstrap.v3.Bootstrap.static_resources>`{.interpreted-text
role="ref"} or have an xDS service such as `LDS
<config_listeners_lds>`{.interpreted-text role="ref"} or
`CDS <config_cluster_manager_cds>`{.interpreted-text role="ref"}
configured in
`dynamic_resources <envoy_v3_api_field_config.bootstrap.v3.Bootstrap.dynamic_resources>`{.interpreted-text
role="ref"}.
