Tracing {#arch_overview_tracing}
=======

Overview
--------

Distributed tracing allows developers to obtain visualizations of call
flows in large service oriented architectures. It can be invaluable in
understanding serialization, parallelism, and sources of latency. Envoy
supports three features related to system wide tracing:

-   **Request ID generation**: Envoy will generate UUIDs when needed and
    populate the
    `config_http_conn_man_headers_x-request-id`{.interpreted-text
    role="ref"} HTTP header. Applications can forward the x-request-id
    header for unified logging as well as tracing. The behavior can be
    configured on per
    `HTTP connection manager<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.request_id_extension>`{.interpreted-text
    role="ref"} basis using an extension.
-   **Client trace ID joining**: The
    `config_http_conn_man_headers_x-client-trace-id`{.interpreted-text
    role="ref"} header can be used to join untrusted request IDs to the
    trusted internal
    `config_http_conn_man_headers_x-request-id`{.interpreted-text
    role="ref"}.
-   **External trace service integration**: Envoy supports pluggable
    external trace visualization providers, that are divided into two
    subgroups:
    -   External tracers which are part of the Envoy code base, like
        [LightStep](https://lightstep.com/),
        [Zipkin](https://zipkin.io/) or any Zipkin compatible backends
        (e.g. [Jaeger](https://github.com/jaegertracing/)), and
        [Datadog](https://datadoghq.com).
    -   External tracers which come as a third party plugin, like
        [Instana](https://www.instana.com/blog/monitoring-envoy-proxy-microservices/).

Support for other tracing providers would not be difficult to add.

How to initiate a trace
-----------------------

The HTTP connection manager that handles the request must have the
`tracing
<envoy_v3_api_msg_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.Tracing>`{.interpreted-text
role="ref"} object set. There are several ways tracing can be initiated:

-   By an external client via the
    `config_http_conn_man_headers_x-client-trace-id`{.interpreted-text
    role="ref"} header.
-   By an internal service via the
    `config_http_conn_man_headers_x-envoy-force-trace`{.interpreted-text
    role="ref"} header.
-   Randomly sampled via the
    `random_sampling <config_http_conn_man_runtime_random_sampling>`{.interpreted-text
    role="ref"} runtime setting.

The router filter is also capable of creating a child span for egress
calls via the
`start_child_span <envoy_v3_api_field_extensions.filters.http.router.v3.Router.start_child_span>`{.interpreted-text
role="ref"} option.

Trace context propagation
-------------------------

Envoy provides the capability for reporting tracing information
regarding communications between services in the mesh. However, to be
able to correlate the pieces of tracing information generated by the
various proxies within a call flow, the services must propagate certain
trace context between the inbound and outbound requests.

Whichever tracing provider is being used, the service should propagate
the `config_http_conn_man_headers_x-request-id`{.interpreted-text
role="ref"} to enable logging across the invoked services to be
correlated.

The tracing providers also require additional context, to enable the
parent/child relationships between the spans (logical units of work) to
be understood. This can be achieved by using the LightStep (via
OpenTracing API) or Zipkin tracer directly within the service itself, to
extract the trace context from the inbound request and inject it into
any subsequent outbound requests. This approach would also enable the
service to create additional spans, describing work being done
internally within the service, that may be useful when examining the
end-to-end trace.

Alternatively the trace context can be manually propagated by the
service:

-   When using the LightStep tracer, Envoy relies on the service to
    propagate the
    `config_http_conn_man_headers_x-ot-span-context`{.interpreted-text
    role="ref"} HTTP header while sending HTTP requests to other
    services.
-   When using the Zipkin tracer, Envoy relies on the service to
    propagate the B3 HTTP headers (
    `config_http_conn_man_headers_x-b3-traceid`{.interpreted-text
    role="ref"},
    `config_http_conn_man_headers_x-b3-spanid`{.interpreted-text
    role="ref"},
    `config_http_conn_man_headers_x-b3-parentspanid`{.interpreted-text
    role="ref"},
    `config_http_conn_man_headers_x-b3-sampled`{.interpreted-text
    role="ref"}, and
    `config_http_conn_man_headers_x-b3-flags`{.interpreted-text
    role="ref"}). The
    `config_http_conn_man_headers_x-b3-sampled`{.interpreted-text
    role="ref"} header can also be supplied by an external client to
    either enable or disable tracing for a particular request. In
    addition, the single
    `config_http_conn_man_headers_b3`{.interpreted-text role="ref"}
    header propagation format is supported, which is a more compressed
    format.
-   When using the Datadog tracer, Envoy relies on the service to
    propagate the Datadog-specific HTTP headers (
    `config_http_conn_man_headers_x-datadog-trace-id`{.interpreted-text
    role="ref"},
    `config_http_conn_man_headers_x-datadog-parent-id`{.interpreted-text
    role="ref"},
    `config_http_conn_man_headers_x-datadog-sampling-priority`{.interpreted-text
    role="ref"}).

What data each trace contains
-----------------------------

An end-to-end trace is comprised of one or more spans. A span represents
a logical unit of work that has a start time and duration and can
contain metadata associated with it. Each span generated by Envoy
contains the following data:

-   Originating service cluster set via
    `--service-cluster`{.interpreted-text role="option"}.
-   Start time and duration of the request.
-   Originating host set via `--service-node`{.interpreted-text
    role="option"}.
-   Downstream cluster set via the
    `config_http_conn_man_headers_downstream-service-cluster`{.interpreted-text
    role="ref"} header.
-   HTTP request URL, method, protocol and user-agent.
-   Additional custom tags set via `custom_tags
    <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.Tracing.custom_tags>`{.interpreted-text
    role="ref"}.
-   Upstream cluster name and address.
-   HTTP response status code.
-   GRPC response status and message (if available).
-   An error tag when HTTP status is 5xx or GRPC status is not \"OK\".
-   Tracing system-specific metadata.

The span also includes a name (or operation) which by default is defined
as the host of the invoked service. However this can be customized using
a `envoy_v3_api_msg_config.route.v3.Decorator`{.interpreted-text
role="ref"} on the route. The name can also be overridden using the
`config_http_filters_router_x-envoy-decorator-operation`{.interpreted-text
role="ref"} header.

Envoy automatically sends spans to tracing collectors. Depending on the
tracing collector, multiple spans are stitched together using common
information such as the globally unique request ID
`config_http_conn_man_headers_x-request-id`{.interpreted-text
role="ref"} (LightStep) or the trace ID configuration (Zipkin and
Datadog). See
`v3 API reference <envoy_v3_api_msg_config.trace.v3.Tracing>`{.interpreted-text
role="ref"} for more information on how to setup tracing in Envoy.

Baggage
-------

Baggage provides a mechanism for data to be available throughout the
entirety of a trace. While metadata such as tags are usually
communicated to collectors out-of-band, baggage data is injected into
the actual request context and available to applications during the
duration of the request. This enables metadata to transparently travel
from the beginning of the request throughout your entire mesh without
relying on application-specific modifications for propagation. See
[OpenTracing\'s
documentation](https://opentracing.io/docs/overview/tags-logs-baggage/)
for more information about baggage.

Tracing providers have varying level of support for getting and setting
baggage:

-   Lightstep (and any OpenTracing-compliant tracer) can read/write
    baggage
-   Zipkin support is not yet implemented
-   X-Ray and OpenCensus don\'t support baggage