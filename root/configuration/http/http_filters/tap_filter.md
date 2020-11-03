Tap {#config_http_filters_tap}
===

-   `v3 API reference <envoy_v3_api_msg_extensions.filters.http.tap.v3.Tap>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.tap*.

::: {.attention}
::: {.title}
Attention
:::

The tap filter is experimental and is currently under active
development. There is currently a very limited set of match conditions,
output configuration, output sinks, etc. Capabilities will be expanded
over time and the configuration structures are likely to change.
:::

The HTTP tap filter is used to interpose on and record HTTP traffic. At
a high level, the configuration is composed of two pieces:

1.  `Match configuration <envoy_v3_api_msg_config.tap.v3.MatchPredicate>`{.interpreted-text
    role="ref"}: a list of conditions under which the filter will match
    an HTTP request and begin a tap session.
2.  `Output configuration <envoy_v3_api_msg_config.tap.v3.OutputConfig>`{.interpreted-text
    role="ref"}: a list of output sinks that the filter will write the
    matched and tapped data to.

Each of these concepts will be covered incrementally over the course of
several example configurations in the following section.

Example configuration
---------------------

Example filter configuration:

``` {.yaml}
name: envoy.filters.http.tap
typed_config:
  "@type": type.googleapis.com/envoy.extensions.filters.http.tap.v3.Tap
  common_config:
    admin_config:
      config_id: test_config_id
```

The previous snippet configures the filter for control via the
:<http:post>:[/tap]{.title-ref} admin handler. See the following section
for more details.

Admin handler {#config_http_filters_tap_admin_handler}
-------------

When the HTTP filter specifies an `admin_config
<envoy_v3_api_msg_extensions.common.tap.v3.AdminConfig>`{.interpreted-text
role="ref"}, it is configured for admin control and the
:<http:post>:[/tap]{.title-ref} admin handler will be installed. The
admin handler can be used for live tapping and debugging of HTTP
traffic. It works as follows:

1.  A POST request is used to provide a valid tap configuration. The
    POST request body can be either the JSON or YAML representation of
    the `TapConfig
    <envoy_v3_api_msg_config.tap.v3.TapConfig>`{.interpreted-text
    role="ref"} message.
2.  If the POST request is accepted, Envoy will stream
    `HttpBufferedTrace
    <envoy_v3_api_msg_data.tap.v3.HttpBufferedTrace>`{.interpreted-text
    role="ref"} messages (serialized to JSON) until the admin request is
    terminated.

An example POST body:

``` {.yaml}
config_id: test_config_id
tap_config:
  match_config:
    and_match:
      rules:
        - http_request_headers_match:
            headers:
              - name: foo
                exact_match: bar
        - http_response_headers_match:
            headers:
              - name: bar
                exact_match: baz
  output_config:
    sinks:
      - streaming_admin: {}
```

The preceding configuration instructs the tap filter to match any HTTP
requests in which a request header `foo: bar` is present AND a response
header `bar: baz` is present. If both of these conditions are met, the
request will be tapped and streamed out the admin endpoint.

Another example POST body:

``` {.yaml}
config_id: test_config_id
tap_config:
  match_config:
    or_match:
      rules:
        - http_request_headers_match:
            headers:
              - name: foo
                exact_match: bar
        - http_response_headers_match:
            headers:
              - name: bar
                exact_match: baz
  output_config:
    sinks:
      - streaming_admin: {}
```

The preceding configuration instructs the tap filter to match any HTTP
requests in which a request header `foo: bar` is present OR a response
header `bar: baz` is present. If either of these conditions are met, the
request will be tapped and streamed out the admin endpoint.

Another example POST body:

``` {.yaml}
config_id: test_config_id
tap_config:
  match_config:
    any_match: true
  output_config:
    sinks:
      - streaming_admin: {}
```

The preceding configuration instructs the tap filter to match any HTTP
requests. All requests will be tapped and streamed out the admin
endpoint.

Another example POST body:

``` {.yaml}
config_id: test_config_id
tap_config:
  match_config:
    and_match:
      rules:
        - http_request_headers_match:
            headers:
              - name: foo
                exact_match: bar
        - http_request_generic_body_match:
            patterns:
              - string_match: test
              - binary_match: 3q2+7w==
            bytes_limit: 128
        - http_response_generic_body_match:
            patterns:
              - binary_match: vu8=
            bytes_limit: 64
  output_config:
    sinks:
      - streaming_admin: {}
```

The preceding configuration instructs the tap filter to match any HTTP
requests in which a request header `foo: bar` is present AND request
body contains string `test` and hex bytes `deadbeef` (`3q2+7w==` in
base64 format) in the first 128 bytes AND response body contains hex
bytes `beef` (`vu8=` in base64 format) in the first 64 bytes. If all of
these conditions are met, the request will be tapped and streamed out to
the admin endpoint.

::: {.attention}
::: {.title}
Attention
:::

Searching for patterns in HTTP body is potentially cpu intensive. For
each specified pattern, http body is scanned byte by byte to find a
match. If multiple patterns are specified, the process is repeated for
each pattern. If location of a pattern is known, `bytes_limit` should be
specified to scan only part of the http body.
:::

Output format
-------------

Each output sink has an associated `format
<envoy_v3_api_enum_config.tap.v3.OutputSink.Format>`{.interpreted-text
role="ref"}. The default format is `JSON_BODY_AS_BYTES
<envoy_v3_api_enum_value_config.tap.v3.OutputSink.Format.JSON_BODY_AS_BYTES>`{.interpreted-text
role="ref"}. This format is easy to read JSON, but has the downside that
body data is base64 encoded. In the case that the tap is known to be on
human readable data, the `JSON_BODY_AS_STRING
<envoy_v3_api_enum_value_config.tap.v3.OutputSink.Format.JSON_BODY_AS_STRING>`{.interpreted-text
role="ref"} format may be more user friendly. See the reference
documentation for more information on other available formats.

An example of a streaming admin tap configuration that uses the
`JSON_BODY_AS_STRING
<envoy_v3_api_enum_value_config.tap.v3.OutputSink.Format.JSON_BODY_AS_STRING>`{.interpreted-text
role="ref"} format:

``` {.yaml}
config_id: test_config_id
tap_config:
  match_config:
    any_match: true
  output_config:
    sinks:
      - format: JSON_BODY_AS_STRING
        streaming_admin: {}
```

Buffered body limits
--------------------

For buffered taps, Envoy will limit the amount of body data that is
tapped to avoid OOM situations. The default limit is 1KiB for both
received (request) and transmitted (response) data. This is configurable
via the `max_buffered_rx_bytes
<envoy_v3_api_field_config.tap.v3.OutputConfig.max_buffered_rx_bytes>`{.interpreted-text
role="ref"} and `max_buffered_tx_bytes
<envoy_v3_api_field_config.tap.v3.OutputConfig.max_buffered_tx_bytes>`{.interpreted-text
role="ref"} settings.

Streaming matching {#config_http_filters_tap_streaming}
------------------

The tap filter supports \"streaming matching.\" This means that instead
of waiting until the end of the request/response sequence, the filter
will match incrementally as the request proceeds. I.e., first the
request headers will be matched, then the request body if present, then
the request trailers if present, then the response headers if present,
etc.

The filter additionally supports optional streamed output which is
governed by the `streaming
<envoy_v3_api_field_config.tap.v3.OutputConfig.streaming>`{.interpreted-text
role="ref"} setting. If this setting is false (the default), Envoy will
emit `fully buffered traces
<envoy_v3_api_msg_data.tap.v3.HttpBufferedTrace>`{.interpreted-text
role="ref"}. Users are likely to find this format easier to interact
with for simple cases.

In cases where fully buffered traces are not practical (e.g., very large
request and responses, long lived streaming APIs, etc.), the streaming
setting can be set to true, and Envoy will emit multiple
`streamed trace segments <envoy_v3_api_msg_data.tap.v3.HttpStreamedTraceSegment>`{.interpreted-text
role="ref"} for each tap. In this case, it is required that
post-processing is performed to stitch all of the trace segments back
together into a usable form. Also note that binary protobuf is not a
self-delimiting format. If binary protobuf output is desired, the
`PROTO_BINARY_LENGTH_DELIMITED
<envoy_v3_api_enum_value_config.tap.v3.OutputSink.Format.PROTO_BINARY_LENGTH_DELIMITED>`{.interpreted-text
role="ref"} output format should be used.

An static filter configuration to enable streaming output looks like:

``` {.yaml}
name: envoy.filters.http.tap
typed_config:
  "@type": type.googleapis.com/envoy.extensions.filters.http.tap.v3.Tap
  common_config:
    static_config:
      match_config:
        http_response_headers_match:
          headers:
            - name: bar
              exact_match: baz
      output_config:
        streaming: true
        sinks:
          - format: PROTO_BINARY_LENGTH_DELIMITED
            file_per_tap:
              path_prefix: /tmp/
```

The previous configuration will match response headers, and as such will
buffer request headers, body, and trailers until a match can be
determined (buffered data limits still apply as described in the
previous section). If a match is determined, buffered data will be
flushed in individual trace segments and then the rest of the tap will
be streamed as data arrives. The messages output might look like this:

``` {.yaml}
http_streamed_trace_segment:
  trace_id: 1
  request_headers:
    headers:
      - key: a
        value: b
```

``` {.yaml}
http_streamed_trace_segment:
  trace_id: 1
  request_body_chunk:
    as_bytes: aGVsbG8=
```

Etc.

Statistics
----------

The tap filter outputs statistics in the *http.\<stat_prefix\>.tap.*
namespace. The `stat prefix
<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stat_prefix>`{.interpreted-text
role="ref"} comes from the owning HTTP connection manager.

  -----------------------------------------------------------------------
  Name              Type              Description
  ----------------- ----------------- -----------------------------------
  rq_tapped         Counter           Total requests that matched and
                                      were tapped

  -----------------------------------------------------------------------
