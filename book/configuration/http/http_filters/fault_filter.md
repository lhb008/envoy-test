Fault Injection {#config_http_filters_fault_injection}
===============

The fault injection filter can be used to test the resiliency of
microservices to different forms of failures. The filter can be used to
inject delays and abort requests with user-specified error codes,
thereby providing the ability to stage different failure scenarios such
as service failures, service overloads, high network latency, network
partitions, etc. Faults injection can be limited to a specific set of
requests based on the (destination) upstream cluster of a request and/or
a set of pre-defined request headers.

The scope of failures is restricted to those that are observable by an
application communicating over the network. CPU and disk failures on the
local host cannot be emulated.

Configuration
-------------

::: {.note}
::: {.title}
Note
:::

The fault injection filter must be inserted before any other filter,
including the router filter.
:::

-   `v3 API reference <envoy_v3_api_msg_extensions.filters.http.fault.v3.HTTPFault>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.fault*.

Controlling fault injection via HTTP headers {#config_http_filters_fault_injection_http_header}
--------------------------------------------

The fault filter has the capability to allow fault configuration to be
specified by the caller. This is useful in certain scenarios in which it
is desired to allow the client to specify its own fault configuration.
The currently supported header controls are:

x-envoy-fault-abort-request

:   HTTP status code to abort a request with. The header value should be
    an integer that specifies the HTTP status code to return in response
    to a request and must be in the range \[200, 600). In order for the
    header to work, `header_abort
    <envoy_v3_api_field_extensions.filters.http.fault.v3.FaultAbort.header_abort>`{.interpreted-text
    role="ref"} needs to be set.

x-envoy-fault-abort-grpc-request

:   gRPC status code to abort a request with. The header value should be
    a non-negative integer that specifies the gRPC status code to return
    in response to a request. Its value range is \[0, UInt32.Max\]
    instead of \[0, 16\] to allow testing even not well-defined gRPC
    status codes. When this header is set, the HTTP response status code
    will be set to 200. In order for the header to work, `header_abort
    <envoy_api_field_config.filter.http.fault.v2.FaultAbort.header_abort>`{.interpreted-text
    role="ref"} needs to be set. If both *x-envoy-fault-abort-request*
    and *x-envoy-fault-abort-grpc-request* headers are set then
    *x-envoy-fault-abort-grpc-request* header will be **ignored** and
    fault response http status code will be set to
    *x-envoy-fault-abort-request* header value.

x-envoy-fault-abort-request-percentage

:   The percentage of requests that should be failed with a status code
    that\'s defined by the value of *x-envoy-fault-abort-request* or
    *x-envoy-fault-abort-grpc-request* HTTP headers. The header value
    should be an integer that specifies the numerator of the percentage
    of request to apply aborts to and must be greater or equal to 0 and
    its maximum value is capped by the value of the numerator of
    `percentage <envoy_v3_api_field_extensions.filters.http.fault.v3.FaultAbort.percentage>`{.interpreted-text
    role="ref"} field. Percentage\'s denominator is equal to default
    percentage\'s denominator
    `percentage <envoy_v3_api_field_extensions.filters.http.fault.v3.FaultAbort.percentage>`{.interpreted-text
    role="ref"} field. In order for the header to work, `header_abort
    <envoy_v3_api_field_extensions.filters.http.fault.v3.FaultAbort.header_abort>`{.interpreted-text
    role="ref"} needs to be set and either *x-envoy-fault-abort-request*
    or *x-envoy-fault-abort-grpc-request* HTTP header needs to be a part
    of the request.

x-envoy-fault-delay-request

:   The duration to delay a request by. The header value should be an
    integer that specifies the number of milliseconds to throttle the
    latency for. In order for the header to work, `header_delay
    <envoy_v3_api_field_extensions.filters.common.fault.v3.FaultDelay.header_delay>`{.interpreted-text
    role="ref"} needs to be set.

x-envoy-fault-delay-request-percentage

:   The percentage of requests that should be delayed by a duration
    that\'s defined by the value of *x-envoy-fault-delay-request* HTTP
    header. The header value should be an integer that specifies the
    percentage of request to apply delays to and must be greater or
    equal to 0 and its maximum value is capped by the value of the
    numerator of
    `percentage <envoy_v3_api_field_extensions.filters.common.fault.v3.FaultDelay.percentage>`{.interpreted-text
    role="ref"} field. Percentage\'s denominator is equal to default
    percentage\'s denominator
    `percentage <envoy_v3_api_field_extensions.filters.common.fault.v3.FaultDelay.percentage>`{.interpreted-text
    role="ref"} field. In order for the header to work, `header_delay
    <envoy_v3_api_field_extensions.filters.common.fault.v3.FaultDelay.header_delay>`{.interpreted-text
    role="ref"} needs to be set and *x-envoy-fault-delay-request* HTTP
    header needs to be a part of a request.

x-envoy-fault-throughput-response

:   The rate limit to use when a response to a caller is sent. The
    header value should be an integer that specifies the limit in KiB/s
    and must be \> 0. In order for the header to work, `header_limit
    <envoy_v3_api_field_extensions.filters.common.fault.v3.FaultRateLimit.header_limit>`{.interpreted-text
    role="ref"} needs to be set.

x-envoy-fault-throughput-response-percentage

:   The percentage of requests whose response rate should be limited to
    the value of *x-envoy-fault-throughput-response* HTTP header. The
    header value should be an integer that specifies the percentage of
    request to apply delays to and must be greater or equal to 0 and its
    maximum value is capped by the value of the numerator of
    `percentage <envoy_v3_api_field_extensions.filters.common.fault.v3.FaultRateLimit.percentage>`{.interpreted-text
    role="ref"} field. Percentage\'s denominator is equal to default
    percentage\'s denominator
    `percentage <envoy_v3_api_field_extensions.filters.common.fault.v3.FaultRateLimit.percentage>`{.interpreted-text
    role="ref"} field. In order for the header to work, `header_limit
    <envoy_v3_api_field_extensions.filters.common.fault.v3.FaultRateLimit.header_limit>`{.interpreted-text
    role="ref"} needs to be set and *x-envoy-fault-delay-request* HTTP
    header needs to be a part of a request.

::: {.attention}
::: {.title}
Attention
:::

Allowing header control is inherently dangerous if exposed to untrusted
clients. In this case, it is suggested to use the `max_active_faults
<envoy_v3_api_field_extensions.filters.http.fault.v3.HTTPFault.max_active_faults>`{.interpreted-text
role="ref"} setting to limit the maximum concurrent faults that can be
active at any given time.
:::

The following is an example configuration that enables header control
for both of the above options:

``` {.yaml}
name: envoy.filters.http.fault
typed_config:
  "@type": type.googleapis.com/envoy.extensions.filters.http.fault.v3.HTTPFault
  max_active_faults: 100
  abort:
    header_abort: {}
    percentage:
      numerator: 100
  delay:
    header_delay: {}
    percentage:
      numerator: 100
  response_rate_limit:
    header_limit: {}
    percentage:
      numerator: 100
```

Runtime {#config_http_filters_fault_injection_runtime}
-------

The HTTP fault injection filter supports the following global runtime
settings:

::: {.attention}
::: {.title}
Attention
:::

Some of the following runtime keys require the filter to be configured
for the specific fault type and some do not. Please consult the
documentation for each key for more information.
:::

fault.http.abort.abort_percent

:   \% of requests that will be aborted if the headers match. Defaults
    to the *abort_percent* specified in config. If the config does not
    contain an *abort* block, then *abort_percent* defaults to 0. For
    historic reasons, this runtime key is available regardless of
    whether the filter is `configured for abort
    <envoy_v3_api_field_extensions.filters.http.fault.v3.HTTPFault.abort>`{.interpreted-text
    role="ref"}.

fault.http.abort.http_status

:   HTTP status code that will be used as the response status code of
    requests that will be aborted if the headers match. Defaults to the
    HTTP status code specified in the config. If the config does not
    contain an *abort* block, then *http_status* defaults to 0. For
    historic reasons, this runtime key is available regardless of
    whether the filter is `configured for abort
    <envoy_v3_api_field_extensions.filters.http.fault.v3.HTTPFault.abort>`{.interpreted-text
    role="ref"}.

fault.http.abort.grpc_status

:   gRPC status code that will be used as the response status code of
    requests that will be aborted if the headers match. Defaults to the
    gRPC status code specified in the config. If this field is missing
    from both the runtime and the config, gRPC status code in the
    response will be derived from *fault.http.abort.http_status* field.
    This runtime key is only available when the filter is
    `configured for abort <envoy_api_field_config.filter.http.fault.v2.HTTPFault.abort>`{.interpreted-text
    role="ref"}.

fault.http.delay.fixed_delay_percent

:   \% of requests that will be delayed if the headers match. Defaults
    to the *delay_percent* specified in the config or 0 otherwise. This
    runtime key is only available when the filter is
    `configured for delay
    <envoy_v3_api_field_extensions.filters.http.fault.v3.HTTPFault.delay>`{.interpreted-text
    role="ref"}.

fault.http.delay.fixed_duration_ms

:   The delay duration in milliseconds. If not specified, the
    *fixed_duration_ms* specified in the config will be used. If this
    field is missing from both the runtime and the config, no delays
    will be injected. This runtime key is only available when the filter
    is `configured for delay
    <envoy_v3_api_field_extensions.filters.http.fault.v3.HTTPFault.delay>`{.interpreted-text
    role="ref"}.

fault.http.max_active_faults

:   The maximum number of active faults (of all types) that Envoy will
    will inject via the fault filter. This can be used in cases where it
    is desired that faults are 100% injected, but the user wants to
    avoid a situation in which too many unexpected concurrent faulting
    requests cause resource constraint issues. If not specified, the
    `max_active_faults
    <envoy_v3_api_field_extensions.filters.http.fault.v3.HTTPFault.max_active_faults>`{.interpreted-text
    role="ref"} setting will be used.

fault.http.rate_limit.response_percent

:   \% of requests which will have a response rate limit fault injected.
    Defaults to the value set in the
    `percentage <envoy_v3_api_field_extensions.filters.common.fault.v3.FaultRateLimit.percentage>`{.interpreted-text
    role="ref"} field. This runtime key is only available when the
    filter is `configured for response rate limiting
    <envoy_v3_api_field_extensions.filters.http.fault.v3.HTTPFault.response_rate_limit>`{.interpreted-text
    role="ref"}.

*Note*, fault filter runtime settings for the specific downstream
cluster override the default ones if present. The following are
downstream specific runtime keys:

-   fault.http.\<downstream-cluster\>.abort.abort_percent
-   fault.http.\<downstream-cluster\>.abort.http_status
-   fault.http.\<downstream-cluster\>.delay.fixed_delay_percent
-   fault.http.\<downstream-cluster\>.delay.fixed_duration_ms

Downstream cluster name is taken from
`the HTTP x-envoy-downstream-service-cluster <config_http_conn_man_headers_downstream-service-cluster>`{.interpreted-text
role="ref"} header. If the following settings are not found in the
runtime it defaults to the global runtime settings which defaults to the
config settings.

Statistics {#config_http_filters_fault_injection_stats}
----------

The fault filter outputs statistics in the *http.\<stat_prefix\>.fault.*
namespace. The `stat prefix
<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stat_prefix>`{.interpreted-text
role="ref"} comes from the owning HTTP connection manager.

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                                     Type              Description
  ---------------------------------------- ----------------- -------------------------------------------------------------------------------------------------------------------------
  delays_injected                          Counter           Total requests that were delayed

  aborts_injected                          Counter           Total requests that were aborted

  response_rl_injected                     Counter           Total requests that had a response rate limit selected for injection (actually injection may not occur due to disconnect,
                                                             reset, no body, etc.)

  faults_overflow                          Counter           Total number of faults that were not injected due to overflowing the
                                                             `max_active_faults <envoy_v3_api_field_extensions.filters.http.fault.v3.HTTPFault.max_active_faults>`{.interpreted-text
                                                             role="ref"} setting

  active_faults                            Gauge             Total number of faults active at the current time

  \<downstream-cluster\>.delays_injected   Counter           Total delayed requests for the given downstream cluster

  \<downstream-cluster\>.aborts_injected   Counter           Total aborted requests for the given downstream cluster
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
