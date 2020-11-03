How do I configure zone aware routing? {#common_configuration_zone_aware_routing}
======================================

There are several steps required for enabling
`zone aware routing <arch_overview_load_balancing_zone_aware_routing>`{.interpreted-text
role="ref"} between source service (\"cluster_a\") and destination
service (\"cluster_b\").

Envoy configuration on the source service
-----------------------------------------

This section describes the specific configuration for the Envoy running
side by side with the source service. These are the requirements:

-   Envoy must be launched with `--service-zone`{.interpreted-text
    role="option"} option which defines the zone for the current host.

-   Both definitions of the source and the destination clusters must
    have
    `EDS <envoy_v3_api_field_config.cluster.v3.Cluster.type>`{.interpreted-text
    role="ref"} type.

-   `local_cluster_name <envoy_v3_api_field_config.bootstrap.v3.ClusterManager.local_cluster_name>`{.interpreted-text
    role="ref"} must be set to the source cluster.

    Only essential parts are listed in the configuration below for the
    cluster manager.

``` {.yaml}
cluster_manager:
  local_cluster_name: cluster_a
static_resources:
  clusters:
  - name: cluster_a
    type: EDS
    eds_cluster_config: ...
  - name: cluster_b
    type: EDS
    eds_cluster_config: ...
```

Envoy configuration on the destination service
----------------------------------------------

It\'s not necessary to run Envoy side by side with the destination
service, but it\'s important that each host in the destination cluster
registers with the discovery service
`queried by the source service Envoy
<config_overview_management_server>`{.interpreted-text role="ref"}.
`Zone <envoy_v3_api_msg_config.endpoint.v3.LocalityLbEndpoints>`{.interpreted-text
role="ref"} information must be available as part of that response.

Only zone related data is listed in the response below.

``` {.yaml}
locality:
  zone: us-east-1d
```

Infrastructure setup
--------------------

The above configuration is necessary for zone aware routing, but there
are certain conditions when zone aware routing is
`not performed <arch_overview_load_balancing_zone_aware_routing_preconditions>`{.interpreted-text
role="ref"}.

Verification steps
------------------

-   Use
    `per zone <config_cluster_manager_cluster_per_az_stats>`{.interpreted-text
    role="ref"} Envoy stats to monitor cross zone traffic.
