Cluster discovery service {#config_cluster_manager_cds}
=========================

The cluster discovery service (CDS) is an optional API that Envoy will
call to dynamically fetch cluster manager members. Envoy will reconcile
the API response and add, modify, or remove known clusters depending on
what is required.

::: {.note}
::: {.title}
Note
:::

Any clusters that are statically defined within the Envoy configuration
cannot be modified or removed via the CDS API.
:::

-   `v3 CDS API <v2_grpc_streaming_endpoints>`{.interpreted-text
    role="ref"}

Statistics
----------

CDS has a `statistics <subscription_statistics>`{.interpreted-text
role="ref"} tree rooted at *cluster_manager.cds.*
