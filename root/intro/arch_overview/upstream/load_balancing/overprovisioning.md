Overprovisioning Factor {#arch_overview_load_balancing_overprovisioning_factor}
=======================

Priority levels and localities are considered overprovisioned with
`this percentage <envoy_v3_api_field_config.endpoint.v3.ClusterLoadAssignment.Policy.overprovisioning_factor>`{.interpreted-text
role="ref"}. Envoy doesn\'t consider a priority level or locality
unavailable until the fraction of available hosts multiplied by the
overprovisioning factor drops below 100. The default value is 140 (in
percentage, which means 140%), so a priority level or locality will not
be considered unavailable until the percentage of available endpoints
goes below 72%.
