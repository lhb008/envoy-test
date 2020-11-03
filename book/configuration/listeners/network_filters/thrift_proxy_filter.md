Thrift proxy {#config_network_filters_thrift_proxy}
============

-   `v3 API reference <envoy_v3_api_msg_extensions.filters.network.thrift_proxy.v3.ThriftProxy>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.network.thrift_proxy*.

Cluster Protocol Options
------------------------

Thrift connections to upstream hosts can be configured by adding an
entry to the appropriate Cluster\'s
`extension_protocol_options<envoy_v3_api_field_config.cluster.v3.Cluster.typed_extension_protocol_options>`{.interpreted-text
role="ref"} keyed by [envoy.filters.network.thrift_proxy]{.title-ref}.
The
`ThriftProtocolOptions<envoy_v3_api_msg_extensions.filters.network.thrift_proxy.v3.ThriftProtocolOptions>`{.interpreted-text
role="ref"} message describes the available options.

Thrift Request Metadata
-----------------------

The
`HEADER transport<envoy_v3_api_enum_value_extensions.filters.network.thrift_proxy.v3.TransportType.HEADER>`{.interpreted-text
role="ref"} and
`TWITTER protocol<envoy_v3_api_enum_value_extensions.filters.network.thrift_proxy.v3.ProtocolType.TWITTER>`{.interpreted-text
role="ref"} support metadata. In particular, the [Header
transport](https://github.com/apache/thrift/blob/master/doc/specs/HeaderFormat.md)
supports informational key/value pairs and the Twitter protocol
transmits [tracing and request context
data](https://github.com/twitter/finagle/blob/master/finagle-thrift/src/main/thrift/tracing.thrift).

### Header Transport Metadata

Header transport key/value pairs are available for routing as
`headers <envoy_v3_api_field_extensions.filters.network.thrift_proxy.v3.RouteMatch.headers>`{.interpreted-text
role="ref"}.

### Twitter Protocol Metadata

Twitter protocol request contexts are converted into headers which are
available for routing as
`headers <envoy_v3_api_field_extensions.filters.network.thrift_proxy.v3.RouteMatch.headers>`{.interpreted-text
role="ref"}. In addition, the following fields are presented as headers:

Client Identifier

:   The ClientId\'s [name]{.title-ref} field (nested in the
    RequestHeader [client_id]{.title-ref} field) becomes the
    [:client-id]{.title-ref} header.

Destination

:   The RequestHeader [dest]{.title-ref} field becomes the
    [:dest]{.title-ref} header.

Delegations

:   Each Delegation from the RequestHeader [delegations]{.title-ref}
    field is added as a header. The header name is the prefix
    [:d:]{.title-ref} followed by the Delegation\'s [src]{.title-ref}.
    The value is the Delegation\'s [dst]{.title-ref} field.

### Metadata Interoperability

Request metadata that is available for routing (see above) is
automatically converted between wire formats when translation between
downstream and upstream connections occurs. Twitter protocol request
contexts, client id, destination, and delegations are therefore
presented as Header transport key/value pairs, named as above.
Similarly, Header transport key/value pairs are presented as Twitter
protocol RequestContext values, unless they match the special names
described above. For instance, a downstream Header transport request
with the info key \":client-id\" is translated to an upstream Twitter
protocol request with a ClientId value.
