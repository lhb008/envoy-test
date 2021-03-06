Circuit breaking {#config_cluster_manager_cluster_circuit_breakers}
================

-   Circuit Breaking
    `architecture overview <arch_overview_circuit_break>`{.interpreted-text
    role="ref"}.
-   `v3 API documentation <envoy_v3_api_msg_config.cluster.v3.CircuitBreakers>`{.interpreted-text
    role="ref"}.

The following is an example circuit breaker configuration:

``` {.yaml}
circuit_breakers:
  thresholds:
  - priority: "DEFAULT"
    max_requests: 75
    max_pending_requests: 35
    retry_budget:
      budget_percent:
        value: 25.0
      min_retry_concurrency: 10
```

Runtime
-------

All circuit breaking settings are runtime configurable for all defined
priorities based on cluster name. They follow the following naming
scheme `circuit_breakers.<cluster_name>.<priority>.<setting>`.
`cluster_name` is the name field in each cluster\'s configuration, which
is set in the Envoy
`config file <envoy_v3_api_field_config.cluster.v3.Cluster.name>`{.interpreted-text
role="ref"}. Available runtime settings will override settings set in
the Envoy config file.
