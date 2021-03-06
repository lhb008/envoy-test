Overload manager {#config_overload_manager}
================

The
`overload manager <arch_overview_overload_manager>`{.interpreted-text
role="ref"} is configured in the Bootstrap
`overload_manager <envoy_v3_api_field_config.bootstrap.v3.Bootstrap.overload_manager>`{.interpreted-text
role="ref"} field.

An example configuration of the overload manager is shown below. It
shows a configuration to disable HTTP/1.x keepalive when heap memory
usage reaches 95% and to stop accepting requests when heap memory usage
reaches 99%.

``` {.yaml}
refresh_interval:
  seconds: 0
  nanos: 250000000
resource_monitors:
  - name: "envoy.resource_monitors.fixed_heap"
    typed_config:
      "@type": type.googleapis.com/envoy.config.resource_monitor.fixed_heap.v2alpha.FixedHeapConfig
      max_heap_size_bytes: 2147483648
actions:
  - name: "envoy.overload_actions.disable_http_keepalive"
    triggers:
      - name: "envoy.resource_monitors.fixed_heap"
        threshold:
          value: 0.95
  - name: "envoy.overload_actions.stop_accepting_requests"
    triggers:
      - name: "envoy.resource_monitors.fixed_heap"
        threshold:
          value: 0.99
```

Resource monitors
-----------------

The overload manager uses Envoy\'s
`extension <extending>`{.interpreted-text role="ref"} framework for
defining resource monitors. Envoy\'s builtin resource monitors are
listed `here <config_resource_monitors>`{.interpreted-text role="ref"}.

Triggers
--------

Triggers connect resource monitors to actions. There are two types of
triggers supported:

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Type                                                                                   Description
  -------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------------------------
  `threshold <envoy_v3_api_msg_config.overload.v3.ThresholdTrigger>`{.interpreted-text   Sets the action state to 1 (= *saturated*) when the resource pressure is above a threshold, and to 0 otherwise.
  role="ref"}                                                                            

  `scaled <envoy_v3_api_msg_config.overload.v3.ScaledTrigger>`{.interpreted-text         Sets the action state to 0 when the resource pressure is below the
  role="ref"}                                                                            `scaling_threshold <envoy_v3_api_field_config.overload.v3.ScaledTrigger.scaling_threshold>`{.interpreted-text
                                                                                         role="ref"}, [(pressure - scaling_threshold)/(saturation_threshold - scaling_threshold)]{.title-ref} when
                                                                                         [scaling_threshold \< pressure \< saturation_threshold]{.title-ref}, and to 1 (*saturated*) when the pressure is
                                                                                         above the
                                                                                         `saturation_threshold <envoy_v3_api_field_config.overload.v3.ScaledTrigger.saturation_threshold>`{.interpreted-text
                                                                                         role="ref"}.\"
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Overload actions {#config_overload_manager_overload_actions}
----------------

The following overload actions are supported:

  ---------------------------------------------------------------------------------------------------
  Name                                                Description
  --------------------------------------------------- -----------------------------------------------
  envoy.overload_actions.stop_accepting_requests      Envoy will immediately respond with a 503
                                                      response code to new requests

  envoy.overload_actions.disable_http_keepalive       Envoy will stop accepting streams on incoming
                                                      HTTP connections

  envoy.overload_actions.stop_accepting_connections   Envoy will stop accepting new network
                                                      connections on its configured listeners

  envoy.overload_actions.shrink_heap                  Envoy will periodically try to shrink the heap
                                                      by releasing free memory to the system
  ---------------------------------------------------------------------------------------------------

Limiting Active Connections
---------------------------

Currently, the only supported way to limit the total number of active
connections allowed across all listeners is via specifying an integer
through the runtime key `overload.global_downstream_max_connections`.
The connection limit is recommended to be less than half of the
system\'s file descriptor limit, to account for upstream connections,
files, and other usage of file descriptors. If the value is unspecified,
there is no global limit on the number of active downstream connections
and Envoy will emit a warning indicating this at startup. To disable the
warning without setting a limit on the number of active downstream
connections, the runtime value may be set to a very large limit (\~2e9).

If it is desired to only limit the number of downstream connections for
a particular listener, per-listener limits can be set via the
`listener configuration <config_listeners>`{.interpreted-text
role="ref"}.

One may simultaneously specify both per-listener and global downstream
connection limits and the conditions will be enforced independently. For
instance, if it is known that a particular listener should have a
smaller number of open connections than others, one may specify a
smaller connection limit for that specific listener and allow the global
limit to enforce resource utilization among all listeners.

An example configuration can be found in the
`edge best practices document <best_practices_edge>`{.interpreted-text
role="ref"}.

Statistics
----------

Each configured resource monitor has a statistics tree rooted at
*overload.\<name\>.* with the following statistics:

  -----------------------------------------------------------------------
  Name              Type              Description
  ----------------- ----------------- -----------------------------------
  pressure          Gauge             Resource pressure as a percent

  failed_updates    Counter           Total failed attempts to update the
                                      resource pressure

  skipped_updates   Counter           Total skipped attempts to update
                                      the resource pressure due to a
                                      pending update
  -----------------------------------------------------------------------

Each configured overload action has a statistics tree rooted at
*overload.\<name\>.* with the following statistics:

  -----------------------------------------------------------------------
  Name              Type              Description
  ----------------- ----------------- -----------------------------------
  active            Gauge             Active state of the action
                                      (0=scaling, 1=saturated)

  scale_percent     Gauge             Scaled value of the action as a
                                      percent (0-99=scaling,
                                      100=saturated)
  -----------------------------------------------------------------------
