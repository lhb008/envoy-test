gRPC HTTP/1.1 bridge {#config_http_filters_grpc_bridge}
====================

-   gRPC `architecture overview <arch_overview_grpc>`{.interpreted-text
    role="ref"}
-   `v3 API reference <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpFilter.name>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.grpc_http1_bridge*.

This is a simple filter which enables the bridging of an HTTP/1.1 client
which does not support response trailers to a compliant gRPC server. It
works by doing the following:

-   When a request is sent, the filter sees if the connection is
    HTTP/1.1 and the request content type is *application/grpc*.
-   If so, when the response is received, the filter buffers it and
    waits for trailers and then checks the *grpc-status* code. If it is
    not zero, the filter switches the HTTP response code to 503. It also
    copies the *grpc-status* and *grpc-message* trailers into the
    response headers so that the client can look at them if it wishes.
-   The client should send HTTP/1.1 requests that translate to the
    following pseudo headers:
    -   *:method*: POST
    -   *:path*: \<gRPC-METHOD-NAME\>
    -   *content-type*: application/grpc
-   The body should be the serialized grpc body which is:
    -   1 byte of zero (not compressed).
    -   network order 4 bytes of proto message length.
    -   serialized proto message.
-   Because this scheme must buffer the response to look for the
    *grpc-status* trailer it will only work with unary gRPC APIs.

This filter also collects stats for all gRPC requests that transit, even
if those requests are normal gRPC requests over HTTP/2.

More info: wire format in [gRPC over
HTTP/2](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md).

::: {.attention}
::: {.title}
Attention
:::

Note that statistics are also collected by the dedicated
`gRPC stats filter
<config_http_filters_grpc_stats>`{.interpreted-text role="ref"}. The use
of this filter for gRPC telemetry has been deprecated.
:::

Statistics
----------

The filter emits statistics in the *cluster.\<route target
cluster\>.grpc.* namespace.

  ------------------------------------------------------------------------
  Name               Type              Description
  ------------------ ----------------- -----------------------------------
  \<grpc             Counter           Total successful service/method
  service\>.\<grpc                     calls
  method\>.success                     

  \<grpc             Counter           Total failed service/method calls
  service\>.\<grpc                     
  method\>.failure                     

  \<grpc             Counter           Total service/method calls
  service\>.\<grpc                     
  method\>.total                       
  ------------------------------------------------------------------------
