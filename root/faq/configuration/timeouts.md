How do I configure timeouts? {#faq_configuration_timeouts}
============================

Envoy supports a wide range of timeouts that may need to be configured
depending on the deployment. This page summarizes the most important
timeouts used in various scenarios.

::: {.attention}
::: {.title}
Attention
:::

This is not an exhaustive list of all of the configurable timeouts that
Envoy supports. Depending on the deployment additional configuration may
be required.
:::

HTTP/gRPC
---------

### Connection timeouts

Connection timeouts apply to the entire HTTP connection and all streams
the connection carries.

-   The HTTP protocol
    `idle timeout <envoy_v3_api_field_config.core.v3.HttpProtocolOptions.idle_timeout>`{.interpreted-text
    role="ref"} is defined in a generic message used by both the HTTP
    connection manager as well as upstream cluster HTTP connections. The
    idle timeout is the time at which a downstream or upstream
    connection will be terminated if there are no active streams. The
    default idle timeout if not otherwise specified is *1 hour*. To
    modify the idle timeout for downstream connections use the
    `common_http_protocol_options
    <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.common_http_protocol_options>`{.interpreted-text
    role="ref"} field in the HTTP connection manager configuration. To
    modify the idle timeout for upstream connections use the
    `common_http_protocol_options <envoy_v3_api_field_config.cluster.v3.Cluster.common_http_protocol_options>`{.interpreted-text
    role="ref"} field in the cluster configuration.

### Stream timeouts

Stream timeouts apply to individual streams carried by an HTTP
connection. Note that a stream is an HTTP/2 and HTTP/3 concept, however
internally Envoy maps HTTP/1 requests to streams so in this context
request/stream is interchangeable.

-   The HTTP connection manager `request_timeout
    <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.request_timeout>`{.interpreted-text
    role="ref"} is the amount of time the connection manager will allow
    for the *entire request stream* to be received from the client.

    ::: {.attention}
    ::: {.title}
    Attention
    :::

    This timeout is not enforced by default as it is not compatible with
    streaming requests (requests that never end). See the stream idle
    timeout that follows. However, if using the
    `buffer filter <config_http_filters_buffer>`{.interpreted-text
    role="ref"}, it is recommended to configure this timeout.
    :::

-   The HTTP connection manager `stream_idle_timeout
    <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stream_idle_timeout>`{.interpreted-text
    role="ref"} is the amount of time that the connection manager will
    allow a stream to exist with no upstream or downstream activity. The
    default stream idle timeout is *5 minutes*. This timeout is strongly
    recommended for all requests (not just streaming requests/responses)
    as it additionally defends against an HTTP/2 peer that does not open
    stream window once an entire response has been buffered to be sent
    to a downstream client.

-   The HTTP protocol
    `max_stream_duration <envoy_v3_api_field_config.core.v3.HttpProtocolOptions.max_stream_duration>`{.interpreted-text
    role="ref"} is defined in a generic message used by the HTTP
    connection manager. The max stream duration is the maximum time that
    a stream\'s lifetime will span. You can use this functionality when
    you want to reset HTTP request/response streams periodically. You
    can\'t use `request_timeout 
    <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.request_timeout>`{.interpreted-text
    role="ref"} in this situation because this timer will be disarmed if
    a response header is received on the request/response streams. This
    timeout is available on both upstream and downstream connections.

### Route timeouts

Envoy supports additional stream timeouts at the route level, as well as
overriding some of the stream timeouts already introduced above.

-   A route
    `timeout <envoy_v3_api_field_config.route.v3.RouteAction.timeout>`{.interpreted-text
    role="ref"} is the amount of time that Envoy will wait for the
    upstream to respond with a complete response. *This timeout does not
    start until the entire downstream request stream has been received*.

    ::: {.attention}
    ::: {.title}
    Attention
    :::

    This timeout defaults to *15 seconds*, however, it is not compatible
    with streaming responses (responses that never end), and will need
    to be disabled. Stream idle timeouts should be used in the case of
    streaming APIs as described elsewhere on this page.
    :::

-   The route
    `idle_timeout <envoy_v3_api_field_config.route.v3.RouteAction.idle_timeout>`{.interpreted-text
    role="ref"} allows overriding of the HTTP connection manager
    `stream_idle_timeout
    <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stream_idle_timeout>`{.interpreted-text
    role="ref"} and does the same thing.

-   The route
    `per_try_timeout <envoy_v3_api_field_config.route.v3.RetryPolicy.per_try_timeout>`{.interpreted-text
    role="ref"} can be configured when using retries so that individual
    tries using a shorter timeout than the overall request timeout
    described above. This timeout only applies before any part of the
    response is sent to the downstream, which normally happens after the
    upstream has sent response headers. This timeout can be used with
    streaming endpoints to retry if the upstream fails to begin a
    response within the timeout.

-   The route
    `MaxStreamDuration proto <envoy_v3_api_msg_config.route.v3.RouteAction.MaxStreamDuration>`{.interpreted-text
    role="ref"} can be used to override the HttpConnectionManager\'s
    `max_stream_duration <envoy_v3_api_field_config.core.v3.HttpProtocolOptions.max_stream_duration>`{.interpreted-text
    role="ref"} for individual routes as well as setting both limits and
    a fixed time offset on grpc-timeout headers.

TCP
---

-   The cluster
    `connect_timeout <envoy_v3_api_field_config.cluster.v3.Cluster.connect_timeout>`{.interpreted-text
    role="ref"} specifies the amount of time Envoy will wait for an
    upstream TCP connection to be established. This timeout has no
    default, but is required in the configuration.

    ::: {.attention}
    ::: {.title}
    Attention
    :::

    For TLS connections, the connect timeout includes the TLS handshake.
    :::

-   The TCP proxy `idle_timeout
    <envoy_v3_api_field_extensions.filters.network.tcp_proxy.v3.TcpProxy.idle_timeout>`{.interpreted-text
    role="ref"} is the amount of time that the TCP proxy will allow a
    connection to exist with no upstream or downstream activity. The
    default idle timeout if not otherwise specified is *1 hour*.
