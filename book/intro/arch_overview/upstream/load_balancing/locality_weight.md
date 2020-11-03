Locality weighted load balancing {#arch_overview_load_balancing_locality_weighted_lb}
================================

One approach to determining how to weight assignments across different
zones and geographical locations is by using explicit weights supplied
via EDS in the
`LocalityLbEndpoints <envoy_v3_api_msg_config.endpoint.v3.LocalityLbEndpoints>`{.interpreted-text
role="ref"} message. This approach is mutually exclusive with
`zone aware routing <arch_overview_load_balancing_zone_aware_routing>`{.interpreted-text
role="ref"}, since in the case of locality aware LB, we rely on the
management server to provide the locality weighting, rather than the
Envoy-side heuristics used in zone aware routing.

When all endpoints are available, the locality is picked using a
weighted round-robin schedule, where the locality weight is used for
weighting. When some endpoints in a locality are unavailable, we adjust
the locality weight to reflect this. As with `priority levels
<arch_overview_load_balancing_priority_levels>`{.interpreted-text
role="ref"}, we assume an
`over-provision factor <arch_overview_load_balancing_overprovisioning_factor>`{.interpreted-text
role="ref"} (default value 1.4), which means we do not perform any
weight adjustment when only a small number of endpoints in a locality
are unavailable.

Assume a simple set-up with 2 localities X and Y, where X has a locality
weight of 1 and Y has a locality weight of 2, L=Y 100% available, with
default overprovisioning factor 1.4.

+-----------------------+----------------------+-----------------------+
| L=X healthy endpoints | Percent of traffic   | Percent of traffic to |
|                       | to L=X               | L=Y                   |
+=======================+======================+=======================+
| 100%                  | 33%                  | > 67%                 |
+-----------------------+----------------------+-----------------------+
| 70%                   | 33%                  | > 67%                 |
+-----------------------+----------------------+-----------------------+
| 69%                   | 32%                  | > 68%                 |
+-----------------------+----------------------+-----------------------+
| 50%                   | 26%                  | > 74%                 |
+-----------------------+----------------------+-----------------------+
| 25%                   | 15%                  | > 85%                 |
+-----------------------+----------------------+-----------------------+
| 0%                    | 0%                   | > 100%                |
+-----------------------+----------------------+-----------------------+

To sum this up in pseudo algorithms:

    availability(L_X) = 140 * available_X_upstreams / total_X_upstreams
    effective_weight(L_X) = locality_weight_X * min(100, availability(L_X))
    load to L_X = effective_weight(L_X) / Σ_c(effective_weight(L_c))

Note that the locality weighted pick takes place after the priority
level is picked. The load balancer follows these steps:

1.  Pick
    `priority level <arch_overview_load_balancing_priority_levels>`{.interpreted-text
    role="ref"}.
2.  Pick locality (as described in this section) within priority level
    from (1).
3.  Pick endpoint using cluster specified load balancer within locality
    from (2).

Locality weighted load balancing is configured by setting
`locality_weighted_lb_config
<envoy_v3_api_field_config.cluster.v3.Cluster.CommonLbConfig.locality_weighted_lb_config>`{.interpreted-text
role="ref"} in the cluster configuration and by providing weights via
`load_balancing_weight
<envoy_v3_api_field_config.endpoint.v3.LocalityLbEndpoints.load_balancing_weight>`{.interpreted-text
role="ref"} and identifying the location of the upstream hosts via
`locality
<envoy_v3_api_field_config.endpoint.v3.LocalityLbEndpoints.locality>`{.interpreted-text
role="ref"} in
`LocalityLbEndpoints <envoy_v3_api_msg_config.endpoint.v3.LocalityLbEndpoints>`{.interpreted-text
role="ref"}.

This feature is not compatible with `load balancer subsetting
<arch_overview_load_balancer_subsets>`{.interpreted-text role="ref"},
since it is not straightforward to reconcile locality level weighting
with sensible weights for individual subsets.
