Versioning
==========

The Envoy xDS APIs follow a well defined
`versioning scheme <api/API_VERSIONING.md>`{.interpreted-text
role="repo"}. Envoy supports
`multiple major versions <api_supported_versions>`{.interpreted-text
role="ref"} at any point in time. The examples in this section are taken
from the v2 xDS API.

Envoy has API versions for both the xDS transport, i.e. the wire
protocol for moving resources between a management server and Envoy, and
for resources. These are known as the transport and resource API version
respectively.

The transport and resource version may be mixed. For example, v3
resources may be transferred over the v2 transport protocol. In
addition, an Envoy may consume mixed resource versions for distinct
resource types. For example,
`v3 Clusters <envoy_v3_api_msg_config.cluster.v3.Cluster>`{.interpreted-text
role="ref"} may be used alongside
`v2 Listeners <envoy_api_msg_Listener>`{.interpreted-text role="ref"}.

Both the transport and resource API versions follow the API versioning
support and deprecation
`policy <api/API_VERSIONING.md>`{.interpreted-text role="repo"}.

::: {.note}
::: {.title}
Note
:::

Envoy will internally operate at the latest xDS resource version and all
supported versioned resources will be transparently upgrading to this
latest version on configuration ingestion. For example, v2 and v3
resources, delivered over either a v2 or v3 transport, or any mix
thereof, will be internally converted into v3 resources.
:::
