Runtime {#config_cluster_manager_cluster_runtime}
=======

Upstream clusters support the following runtime settings:

Active health checking
----------------------

health_check.min_interval

:   Min value for the health checking
    `interval <envoy_v3_api_field_config.core.v3.HealthCheck.interval>`{.interpreted-text
    role="ref"}. Default value is 1 ms. The effective health check
    interval will be no less than 1ms. The health checking interval will
    be between *min_interval* and *max_interval*.

health_check.max_interval

:   Max value for the health checking
    `interval <envoy_v3_api_field_config.core.v3.HealthCheck.interval>`{.interpreted-text
    role="ref"}. Default value is MAX_INT. The effective health check
    interval will be no less than 1ms. The health checking interval will
    be between *min_interval* and *max_interval*.

health_check.verify_cluster

:   What % of health check requests will be verified against the
    `expected upstream service
    <envoy_v3_api_field_config.core.v3.HealthCheck.HttpHealthCheck.service_name_matcher>`{.interpreted-text
    role="ref"} as the `health check filter
    <arch_overview_health_checking_filter>`{.interpreted-text
    role="ref"} will write the remote service cluster into the response.

Outlier detection {#config_cluster_manager_cluster_runtime_outlier_detection}
-----------------

See the outlier detection
`architecture overview <arch_overview_outlier_detection>`{.interpreted-text
role="ref"} for more information on outlier detection. The runtime
parameters supported by outlier detection are the same as the
`static configuration parameters <envoy_v3_api_msg_config.cluster.v3.OutlierDetection>`{.interpreted-text
role="ref"}, namely:

outlier_detection.consecutive_5xx

:   `consecutive_5XX
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.consecutive_5xx>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.consecutive_gateway_failure

:   `consecutive_gateway_failure
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.consecutive_gateway_failure>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.consecutive_local_origin_failure

:   `consecutive_local_origin_failure
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.consecutive_local_origin_failure>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.interval_ms

:   `interval_ms
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.interval>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.base_ejection_time_ms

:   `base_ejection_time_ms
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.base_ejection_time>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.max_ejection_percent

:   `max_ejection_percent
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.max_ejection_percent>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.enforcing_consecutive_5xx

:   `enforcing_consecutive_5xx
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.enforcing_consecutive_5xx>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.enforcing_consecutive_gateway_failure

:   `enforcing_consecutive_gateway_failure
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.enforcing_consecutive_gateway_failure>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.enforcing_consecutive_local_origin_failure

:   `enforcing_consecutive_local_origin_failure
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.enforcing_consecutive_local_origin_failure>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.enforcing_success_rate

:   `enforcing_success_rate
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.enforcing_success_rate>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.enforcing_local_origin_success_rate

:   `enforcing_local_origin_success_rate
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.enforcing_local_origin_success_rate>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.success_rate_minimum_hosts

:   `success_rate_minimum_hosts
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.success_rate_minimum_hosts>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.success_rate_request_volume

:   `success_rate_request_volume
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.success_rate_request_volume>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.success_rate_stdev_factor

:   `success_rate_stdev_factor
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.success_rate_stdev_factor>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.enforcing_failure_percentage

:   `enforcing_failure_percentage
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.enforcing_failure_percentage>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.enforcing_failure_percentage_local_origin

:   `enforcing_failure_percentage_local_origin
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.enforcing_failure_percentage_local_origin>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.failure_percentage_request_volume

:   `failure_percentage_request_volume
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.failure_percentage_request_volume>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.failure_percentage_minimum_hosts

:   `failure_percentage_minimum_hosts
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.failure_percentage_minimum_hosts>`{.interpreted-text
    role="ref"} setting in outlier detection

outlier_detection.failure_percentage_threshold

:   `failure_percentage_threshold
    <envoy_v3_api_field_config.cluster.v3.OutlierDetection.failure_percentage_threshold>`{.interpreted-text
    role="ref"} setting in outlier detection

Core
----

upstream.healthy_panic_threshold

:   Sets the
    `panic threshold <arch_overview_load_balancing_panic_threshold>`{.interpreted-text
    role="ref"} percentage. Defaults to 50%.

upstream.use_http2

:   Whether the cluster utilizes the *http2*
    `protocol options <envoy_v3_api_field_config.cluster.v3.Cluster.http2_protocol_options>`{.interpreted-text
    role="ref"} if configured. Set to 0 to disable HTTP/2 even if the
    feature is configured. Defaults to enabled.

Zone aware load balancing {#config_cluster_manager_cluster_runtime_zone_routing}
-------------------------

upstream.zone_routing.enabled

:   \% of requests that will be routed to the same upstream zone.
    Defaults to 100% of requests.

upstream.zone_routing.min_cluster_size

:   Minimal size of the upstream cluster for which zone aware routing
    can be attempted. Default value is 6. If the upstream cluster size
    is smaller than *min_cluster_size* zone aware routing will not be
    performed.

Circuit breaking
----------------

circuit_breakers.\<cluster_name\>.\<priority\>.max_connections

:   `Max connections circuit breaker setting <envoy_v3_api_field_config.cluster.v3.CircuitBreakers.Thresholds.max_connections>`{.interpreted-text
    role="ref"}

circuit_breakers.\<cluster_name\>.\<priority\>.max_pending_requests

:   `Max pending requests circuit breaker setting <envoy_v3_api_field_config.cluster.v3.CircuitBreakers.Thresholds.max_pending_requests>`{.interpreted-text
    role="ref"}

circuit_breakers.\<cluster_name\>.\<priority\>.max_requests

:   `Max requests circuit breaker setting <envoy_v3_api_field_config.cluster.v3.CircuitBreakers.Thresholds.max_requests>`{.interpreted-text
    role="ref"}

circuit_breakers.\<cluster_name\>.\<priority\>.max_retries

:   `Max retries circuit breaker setting <envoy_v3_api_field_config.cluster.v3.CircuitBreakers.Thresholds.max_retries>`{.interpreted-text
    role="ref"}

circuit_breakers.\<cluster_name\>.\<priority\>.retry_budget.budget_percent

:   `Max retries circuit breaker setting <envoy_v3_api_field_config.cluster.v3.CircuitBreakers.Thresholds.RetryBudget.budget_percent>`{.interpreted-text
    role="ref"}

circuit_breakers.\<cluster_name\>.\<priority\>.retry_budget.min_retry_concurrency

:   `Max retries circuit breaker setting <envoy_v3_api_field_config.cluster.v3.CircuitBreakers.Thresholds.RetryBudget.min_retry_concurrency>`{.interpreted-text
    role="ref"}
