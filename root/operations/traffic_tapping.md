Traffic tapping {#operations_traffic_tapping}
===============

Envoy currently provides two experimental extensions that can tap
traffic:

> -   `HTTP tap filter <config_http_filters_tap>`{.interpreted-text
>     role="ref"}. See the linked filter documentation for more
>     information.
> -   `Tap transport socket extension <envoy_v3_api_msg_config.core.v3.TransportSocket>`{.interpreted-text
>     role="ref"} that can intercept traffic and write to a
>     `protobuf trace file
>     <envoy_v3_api_msg_data.tap.v3.TraceWrapper>`{.interpreted-text
>     role="ref"}. The remainder of this document describes the
>     configuration of the tap transport socket.

Tap transport socket configuration
----------------------------------

::: {.attention}
::: {.title}
Attention
:::

The tap transport socket is experimental and is currently under active
development. There is currently a very limited set of match conditions,
output configuration, output sinks, etc. Capabilities will be expanded
over time and the configuration structures are likely to change.
:::

Tapping can be configured on `Listener
<envoy_v3_api_field_config.listener.v3.FilterChain.transport_socket>`{.interpreted-text
role="ref"} and `Cluster
<envoy_v3_api_field_config.cluster.v3.Cluster.transport_socket>`{.interpreted-text
role="ref"} transport sockets, providing the ability to interpose on
downstream and upstream L4 connections respectively.

To configure traffic tapping, add an
[envoy.transport_sockets.tap]{.title-ref} transport socket
`configuration <envoy_v3_api_msg_extensions.filters.http.tap.v3.Tap>`{.interpreted-text
role="ref"} to the listener or cluster. For a plain text socket this
might look like:

``` {.yaml}
transport_socket:
  name: envoy.transport_sockets.tap
  typed_config:
    "@type": type.googleapis.com/envoy.extensions.transport_sockets.tap.v3.Tap
    common_config:
      static_config:
        match_config:
          any_match: true
        output_config:
          sinks:
            - format: PROTO_BINARY
              file_per_tap:
                path_prefix: /some/tap/path
    transport_socket:
      name: envoy.transport_sockets.raw_buffer
```

For a TLS socket, this will be:

``` {.yaml}
transport_socket:
  name: envoy.transport_sockets.tap
  typed_config:
    "@type": type.googleapis.com/envoy.extensions.transport_sockets.tap.v3.Tap
    common_config:
      static_config:
        match_config:
          any_match: true
        output_config:
          sinks:
            - format: PROTO_BINARY
              file_per_tap:
                path_prefix: /some/tap/path
    transport_socket:
      name: envoy.transport_sockets.tls
      typed_config: <TLS context>
```

where the TLS context configuration replaces any existing `downstream
<envoy_v3_api_msg_extensions.transport_sockets.tls.v3.DownstreamTlsContext>`{.interpreted-text
role="ref"} or `upstream
<envoy_v3_api_msg_extensions.transport_sockets.tls.v3.UpstreamTlsContext>`{.interpreted-text
role="ref"} TLS configuration on the listener or cluster, respectively.

Each unique socket instance will generate a trace file prefixed with
[path_prefix]{.title-ref}. E.g. [/some/tap/path_0.pb]{.title-ref}.

Buffered data limits
--------------------

For buffered socket taps, Envoy will limit the amount of body data that
is tapped to avoid OOM situations. The default limit is 1KiB for both
received and transmitted data. This is configurable via the
`max_buffered_rx_bytes
<envoy_v3_api_field_config.tap.v3.OutputConfig.max_buffered_rx_bytes>`{.interpreted-text
role="ref"} and `max_buffered_tx_bytes
<envoy_v3_api_field_config.tap.v3.OutputConfig.max_buffered_tx_bytes>`{.interpreted-text
role="ref"} settings. When a buffered socket tap is truncated, the trace
will indicate truncation via the `read_truncated
<envoy_v3_api_field_data.tap.v3.SocketBufferedTrace.read_truncated>`{.interpreted-text
role="ref"} and `write_truncated
<envoy_v3_api_field_data.tap.v3.SocketBufferedTrace.write_truncated>`{.interpreted-text
role="ref"} fields as well as the body
`truncated <envoy_v3_api_field_data.tap.v3.Body.truncated>`{.interpreted-text
role="ref"} field.

Streaming
---------

The tap transport socket supports both buffered and streaming,
controlled by the `streaming
<envoy_v3_api_field_config.tap.v3.OutputConfig.streaming>`{.interpreted-text
role="ref"} setting. When buffering,
`SocketBufferedTrace <envoy_v3_api_msg_data.tap.v3.SocketBufferedTrace>`{.interpreted-text
role="ref"} messages are emitted. When streaming, a series of
`SocketStreamedTraceSegment
<envoy_v3_api_msg_data.tap.v3.SocketStreamedTraceSegment>`{.interpreted-text
role="ref"} are emitted.

See the
`HTTP tap filter streaming <config_http_filters_tap_streaming>`{.interpreted-text
role="ref"} documentation for more information. Most of the concepts
overlap between the HTTP filter and the transport socket.

PCAP generation
---------------

The generated trace file can be converted to [libpcap
format](https://wiki.wireshark.org/Development/LibpcapFileFormat),
suitable for analysis with tools such as
[Wireshark](https://www.wireshark.org/) with the [tap2pcap]{.title-ref}
utility, e.g.:

``` {.bash}
bazel run @envoy_api_canonical//tools:tap2pcap /some/tap/path_0.pb path_0.pcap
tshark -r path_0.pcap -d "tcp.port==10000,http2" -P
  1   0.000000    127.0.0.1 → 127.0.0.1    HTTP2 157 Magic, SETTINGS, WINDOW_UPDATE, HEADERS
  2   0.013713    127.0.0.1 → 127.0.0.1    HTTP2 91 SETTINGS, SETTINGS, WINDOW_UPDATE
  3   0.013820    127.0.0.1 → 127.0.0.1    HTTP2 63 SETTINGS
  4   0.128649    127.0.0.1 → 127.0.0.1    HTTP2 5586 HEADERS
  5   0.130006    127.0.0.1 → 127.0.0.1    HTTP2 7573 DATA
  6   0.131044    127.0.0.1 → 127.0.0.1    HTTP2 3152 DATA, DATA
```
