Introduction
============

The Envoy xDS APIs are defined as
[proto3](https://developers.google.com/protocol-buffers/docs/proto3)
[Protocol Buffers](https://developers.google.com/protocol-buffers/) in
the `api tree <api/>`{.interpreted-text role="repo"}. They support:

-   Streaming delivery of `xDS <xds_protocol>`{.interpreted-text
    role="ref"} API updates via gRPC. This reduces resource requirements
    and can lower the update latency.
-   A new REST-JSON API in which the JSON/YAML formats are derived
    mechanically via the [proto3 canonical JSON
    mapping](https://developers.google.com/protocol-buffers/docs/proto3#json).
-   Delivery of updates via the filesystem, REST-JSON or gRPC endpoints.
-   Advanced load balancing through an extended endpoint assignment API
    and load and resource utilization reporting to management servers.
-   `Stronger consistency and ordering properties
    <xds_protocol_eventual_consistency_considerations>`{.interpreted-text
    role="ref"} when needed. The APIs still maintain a baseline eventual
    consistency model.

See the `xDS protocol description <xds_protocol>`{.interpreted-text
role="ref"} for further details on aspects of xDS message exchange
between Envoy and the management server.
