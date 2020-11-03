Administration interface {#operations_admin_interface}
========================

Envoy exposes a local administration interface that can be used to query
and modify different aspects of the server:

-   `v3 API reference <envoy_v3_api_msg_config.bootstrap.v3.Admin>`{.interpreted-text
    role="ref"}

::: {#operations_admin_interface_security}
::: {.attention}
::: {.title}
Attention
:::

The administration interface in its current form both allows destructive
operations to be performed (e.g., shutting down the server) as well as
potentially exposes private information (e.g., stats, cluster names,
cert info, etc.). It is **critical** that access to the administration
interface is only allowed via a secure network. It is also **critical**
that hosts that access the administration interface are **only**
attached to the secure network (i.e., to avoid CSRF attacks). This
involves setting up an appropriate firewall or optimally only allowing
access to the administration listener via localhost. This can be
accomplished with a v2 configuration like the following:

``` {.yaml}
admin:
  access_log_path: /tmp/admin_access.log
  profile_path: /tmp/envoy.prof
  address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }
```

In the future additional security options will be added to the
administration interface. This work is tracked in
[this](https://github.com/envoyproxy/envoy/issues/2763) issue.

All mutations must be sent as HTTP POST operations. When a mutation is
requested via GET, the request has no effect, and an HTTP 400 (Invalid
Request) response is returned.
:::
:::

::: {.note}
::: {.title}
Note
:::

For an endpoint with *?format=json*, it dumps data as a JSON-serialized
proto. Fields with default values are not rendered. For example for
*/clusters?format=json*, the circuit breakers thresholds priority field
is omitted when its value is `DEFAULT priority
<envoy_v3_api_enum_value_config.core.v3.RoutingPriority.DEFAULT>`{.interpreted-text
role="ref"} as shown below:

``` {.json}
{
 "thresholds": [
  {
   "max_connections": 1,
   "max_pending_requests": 1024,
   "max_requests": 1024,
   "max_retries": 1
  },
  {
   "priority": "HIGH",
   "max_connections": 1,
   "max_pending_requests": 1024,
   "max_requests": 1024,
   "max_retries": 1
  }
 ]
}
```
:::

::: {#operations_admin_interface_certs}
:::

::: {#operations_admin_interface_clusters}
:::

::: {#operations_admin_interface_config_dump}
:::

::: {.warning}
::: {.title}
Warning
:::

Configuration may include
`TLS certificates <envoy_v3_api_msg_extensions.transport_sockets.tls.v3.TlsCertificate>`{.interpreted-text
role="ref"}. Before dumping the configuration, Envoy will attempt to
redact the `private_key` and `password` fields from any certificates it
finds. This relies on the configuration being a strongly-typed protobuf
message. If your Envoy configuration uses deprecated `config` fields (of
type `google.protobuf.Struct`), please update to the recommended
`typed_config` fields (of type `google.protobuf.Any`) to ensure
sensitive data is redacted properly.
:::

::: {.warning}
::: {.title}
Warning
:::

The underlying proto is marked v2alpha and hence its contents, including
the JSON representation, are not guaranteed to be stable.
:::

::: {#operations_admin_interface_config_dump_include_eds}
:::

::: {#operations_admin_interface_config_dump_by_mask}
:::

::: {#operations_admin_interface_config_dump_by_resource}
:::

::: {#operations_admin_interface_config_dump_by_resource_and_mask}
:::

::: {#operations_admin_interface_healthcheck_fail}
:::

::: {#operations_admin_interface_healthcheck_ok}
:::

::: {#operations_admin_interface_init_dump}
:::

::: {#operations_admin_interface_init_dump_by_mask}
:::

::: {#operations_admin_interface_listeners}
:::

::: {#operations_admin_interface_logging}
:::

::: {#operations_admin_interface_drain}
:::

::: {.attention}
::: {.title}
Attention
:::

This operation directly stops the matched listeners on workers. Once
listeners in a given traffic direction are stopped, listener additions
and modifications in that direction are not allowed.
:::

::: {#operations_admin_interface_stats}
:::

::: {#operations_admin_interface_runtime}
:::

``` {.json}
{
  "layers": [
    "disk",
    "override",
    "admin",
  ],
  "entries": {
    "my_key": {
      "layer_values": [
        "my_disk_value",
        "",
        ""
      ],
      "final_value": "my_disk_value"
    },
    "my_second_key": {
      "layer_values": [
        "my_second_disk_value",
        "my_disk_override_value",
        "my_admin_override_value"
      ],
      "final_value": "my_admin_override_value"
    }
  }
}
```

::: {#operations_admin_interface_runtime_modify}
:::

::: {.attention}
::: {.title}
Attention
:::

Use the /runtime_modify endpoint with care. Changes are effectively
immediately. It is **critical** that the admin interface is
`properly secured
<operations_admin_interface_security>`{.interpreted-text role="ref"}.
:::
