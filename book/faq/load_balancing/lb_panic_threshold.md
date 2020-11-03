I setup health checking. When I fail some hosts, Envoy starts routing to all of them again. Why?
================================================================================================

This feature is known as the load balancer `panic threshold
<arch_overview_load_balancing_panic_threshold>`{.interpreted-text
role="ref"}. It is used to prevent cascading failure when upstream hosts
start failing health checks in large numbers.
