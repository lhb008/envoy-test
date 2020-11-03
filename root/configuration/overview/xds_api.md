xDS API endpoints {#config_overview_management_server}
=================

An xDS management server will implement the below endpoints as required
for gRPC and/or REST serving. In both streaming gRPC and REST-JSON
cases, a
`DiscoveryRequest <envoy_v3_api_msg_service.discovery.v3.DiscoveryRequest>`{.interpreted-text
role="ref"} is sent and a
`DiscoveryResponse <envoy_v3_api_msg_service.discovery.v3.DiscoveryResponse>`{.interpreted-text
role="ref"} received following the
`xDS protocol <xds_protocol>`{.interpreted-text role="ref"}.

Below we describe endpoints for the v2 and v3 transport API versions.

gRPC streaming endpoints {#v2_grpc_streaming_endpoints}
------------------------

See `cds.proto <api/service/cluster/v3/cds.proto>`{.interpreted-text
role="repo"} for the service definition. This is used by Envoy as a
client when

``` {.yaml}
cds_config:
  api_config_source:
    api_type: GRPC
    transport_api_version: <V2|V3>
    grpc_services:
      envoy_grpc:
        cluster_name: some_xds_cluster
```

is set in the `dynamic_resources
<envoy_v3_api_field_config.bootstrap.v3.Bootstrap.dynamic_resources>`{.interpreted-text
role="ref"} of the `Bootstrap
<envoy_v3_api_msg_config.bootstrap.v3.Bootstrap>`{.interpreted-text
role="ref"} config.

See `eds.proto
<api/envoy/service/endpoint/v3/eds.proto>`{.interpreted-text
role="repo"} for the service definition. This is used by Envoy as a
client when

``` {.yaml}
eds_config:
  api_config_source:
    api_type: GRPC
    transport_api_version: <V2|V3>
    grpc_services:
      envoy_grpc:
        cluster_name: some_xds_cluster
```

is set in the `eds_cluster_config
<envoy_v3_api_field_config.cluster.v3.Cluster.eds_cluster_config>`{.interpreted-text
role="ref"} field of the `Cluster
<envoy_v3_api_msg_config.cluster.v3.Cluster>`{.interpreted-text
role="ref"} config.

See `lds.proto
<api/envoy/service/listener/v3/lds.proto>`{.interpreted-text
role="repo"} for the service definition. This is used by Envoy as a
client when

``` {.yaml}
lds_config:
  api_config_source:
    api_type: GRPC
    transport_api_version: <V2|V3>
    grpc_services:
      envoy_grpc:
        cluster_name: some_xds_cluster
```

is set in the `dynamic_resources
<envoy_v3_api_field_config.bootstrap.v3.Bootstrap.dynamic_resources>`{.interpreted-text
role="ref"} of the `Bootstrap
<envoy_v3_api_msg_config.bootstrap.v3.Bootstrap>`{.interpreted-text
role="ref"} config.

See `rds.proto
<api/envoy/service/route/v3/rds.proto>`{.interpreted-text role="repo"}
for the service definition. This is used by Envoy as a client when

``` {.yaml}
route_config_name: some_route_name
config_source:
  api_config_source:
    api_type: GRPC
    transport_api_version: <V2|V3>
    grpc_services:
      envoy_grpc:
        cluster_name: some_xds_cluster
```

is set in the `rds
<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.rds>`{.interpreted-text
role="ref"} field of the `HttpConnectionManager
<envoy_v3_api_msg_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager>`{.interpreted-text
role="ref"} config.

See `srds.proto
<api/envoy/service/route/v3/srds.proto>`{.interpreted-text role="repo"}
for the service definition. This is used by Envoy as a client when

``` {.yaml}
name: some_scoped_route_name
scoped_rds:
  config_source:
    api_config_source:
      api_type: GRPC
      transport_api_version: <V2|V3>
      grpc_services:
        envoy_grpc:
          cluster_name: some_xds_cluster
```

is set in the `scoped_routes
<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.scoped_routes>`{.interpreted-text
role="ref"} field of the `HttpConnectionManager
<envoy_v3_api_msg_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager>`{.interpreted-text
role="ref"} config.

See `sds.proto
<api/envoy/service/secret/v3/sds.proto>`{.interpreted-text role="repo"}
for the service definition. This is used by Envoy as a client when

``` {.yaml}
name: some_secret_name
config_source:
  api_config_source:
    api_type: GRPC
    transport_api_version: <V2|V3>
    grpc_services:
      envoy_grpc:
        cluster_name: some_xds_cluster
```

is set inside a
`SdsSecretConfig <envoy_v3_api_msg_extensions.transport_sockets.tls.v3.SdsSecretConfig>`{.interpreted-text
role="ref"} message. This message is used in various places such as the
`CommonTlsContext <envoy_v3_api_msg_extensions.transport_sockets.tls.v3.CommonTlsContext>`{.interpreted-text
role="ref"}.

See `rtds.proto
<api/envoy/service/runtime/v3/rtds.proto>`{.interpreted-text
role="repo"} for the service definition. This is used by Envoy as a
client when

``` {.yaml}
name: some_runtime_layer_name
config_source:
  api_config_source:
    api_type: GRPC
    transport_api_version: <V2|V3>
    grpc_services:
      envoy_grpc:
        cluster_name: some_xds_cluster
```

is set inside the
`rtds_layer <envoy_v3_api_field_config.bootstrap.v3.RuntimeLayer.rtds_layer>`{.interpreted-text
role="ref"} field.

REST endpoints
--------------

See `cds.proto
<api/envoy/service/cluster/v3/cds.proto>`{.interpreted-text role="repo"}
for the service definition. This is used by Envoy as a client when

``` {.yaml}
cds_config:
  api_config_source:
    api_type: REST
    transport_api_version: <V2|V3>
    cluster_names: [some_xds_cluster]
```

is set in the `dynamic_resources
<envoy_v3_api_field_config.bootstrap.v3.Bootstrap.dynamic_resources>`{.interpreted-text
role="ref"} of the `Bootstrap
<envoy_v3_api_msg_config.bootstrap.v3.Bootstrap>`{.interpreted-text
role="ref"} config.

See `eds.proto
<api/envoy/service/endpoint/v3/eds.proto>`{.interpreted-text
role="repo"} for the service definition. This is used by Envoy as a
client when

``` {.yaml}
eds_config:
  api_config_source:
    api_type: REST
    transport_api_version: <V2|V3>
    cluster_names: [some_xds_cluster]
```

is set in the `eds_cluster_config
<envoy_v3_api_field_config.cluster.v3.Cluster.eds_cluster_config>`{.interpreted-text
role="ref"} field of the `Cluster
<envoy_v3_api_msg_config.cluster.v3.Cluster>`{.interpreted-text
role="ref"} config.

See `lds.proto
<api/envoy/service/listener/v3/lds.proto>`{.interpreted-text
role="repo"} for the service definition. This is used by Envoy as a
client when

``` {.yaml}
lds_config:
  api_config_source:
    api_type: REST
    transport_api_version: <V2|V3>
    cluster_names: [some_xds_cluster]
```

is set in the `dynamic_resources
<envoy_v3_api_field_config.bootstrap.v3.Bootstrap.dynamic_resources>`{.interpreted-text
role="ref"} of the `Bootstrap
<envoy_v3_api_msg_config.bootstrap.v3.Bootstrap>`{.interpreted-text
role="ref"} config.

See `rds.proto
<api/envoy/service/route/v3/rds.proto>`{.interpreted-text role="repo"}
for the service definition. This is used by Envoy as a client when

``` {.yaml}
route_config_name: some_route_name
config_source:
  api_config_source:
    api_type: REST
    transport_api_version: <V2|V3>
    cluster_names: [some_xds_cluster]
```

is set in the `rds
<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.rds>`{.interpreted-text
role="ref"} field of the `HttpConnectionManager
<envoy_v3_api_msg_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager>`{.interpreted-text
role="ref"} config.

::: {.note}
::: {.title}
Note
:::

The management server responding to these endpoints must respond with a
`DiscoveryResponse <envoy_api_msg_DiscoveryResponse>`{.interpreted-text
role="ref"} along with a HTTP status of 200. Additionally, if the
configuration that would be supplied has not changed (as indicated by
the version supplied by the Envoy client) then the management server can
respond with an empty body and a HTTP status of 304.
:::

Aggregated Discovery Service {#config_overview_ads}
----------------------------

While Envoy fundamentally employs an eventual consistency model, ADS
provides an opportunity to sequence API update pushes and ensure
affinity of a single management server for an Envoy node for API
updates. ADS allows one or more APIs and their resources to be delivered
on a single, bidirectional gRPC stream by the management server. Without
this, some APIs such as RDS and EDS may require the management of
multiple streams and connections to distinct management servers.

ADS will allow for hitless updates of configuration by appropriate
sequencing. For example, suppose *foo.com* was mapped to cluster *X*. We
wish to change the mapping in the route table to point *foo.com* at
cluster *Y*. In order to do this, a CDS/EDS update must first be
delivered containing both clusters *X* and *Y*.

Without ADS, the CDS/EDS/RDS streams may point at distinct management
servers, or when on the same management server at distinct gRPC
streams/connections that require coordination. The EDS resource requests
may be split across two distinct streams, one for *X* and one for *Y*.
ADS allows these to be coalesced to a single stream to a single
management server, avoiding the need for distributed synchronization to
correctly sequence the update. With ADS, the management server would
deliver the CDS, EDS and then RDS updates on a single stream.

ADS is only available for gRPC streaming (not REST) and is described
more fully in `xDS <xds_protocol_ads>`{.interpreted-text role="ref"}
document. The gRPC endpoint is:

See `discovery.proto
<api/envoy/service/discovery/v3/discovery.proto>`{.interpreted-text
role="repo"} for the service definition. This is used by Envoy as a
client when

``` {.yaml}
ads_config:
  api_type: GRPC
  transport_api_version: <V2|V3>
  grpc_services:
    envoy_grpc:
      cluster_name: some_ads_cluster
```

is set in the `dynamic_resources
<envoy_v3_api_field_config.bootstrap.v3.Bootstrap.dynamic_resources>`{.interpreted-text
role="ref"} of the `Bootstrap
<envoy_v3_api_msg_config.bootstrap.v3.Bootstrap>`{.interpreted-text
role="ref"} config.

When this is set, any of the configuration sources
`above <v2_grpc_streaming_endpoints>`{.interpreted-text role="ref"} can
be set to use the ADS channel. For example, a LDS config could be
changed from

``` {.yaml}
lds_config:
  api_config_source:
    api_type: REST
    cluster_names: [some_xds_cluster]
```

to

``` {.yaml}
lds_config: {ads: {}}
```

with the effect that the LDS stream will be directed to
*some_ads_cluster* over the shared ADS channel.

Delta endpoints {#config_overview_delta}
---------------

The REST, filesystem, and original gRPC xDS implementations all deliver
\"state of the world\" updates: every CDS update must contain every
cluster, with the absence of a cluster from an update implying that the
cluster is gone. For Envoy deployments with huge amounts of resources
and even a trickle of churn, these state-of-the-world updates can be
cumbersome.

As of 1.12.0, Envoy supports a \"delta\" variant of xDS (including ADS),
where updates only contain resources added/changed/removed. Delta xDS is
a gRPC (only) protocol. Delta uses different request/response protos
than SotW (DeltaDiscovery{Request,Response}); see
`discovery.proto <api/envoy/service/discovery/v3/discovery.proto>`{.interpreted-text
role="repo"}. Conceptually, delta should be viewed as a new xDS
transport type: there is static, filesystem, REST, gRPC-SotW, and now
gRPC-delta. (Envoy\'s implementation of the gRPC-SotW/delta client
happens to share most of its code between the two, and something similar
is likely possible on the server side. However, they are in fact
incompatible protocols.
`The specification of the delta xDS protocol's behavior is here <xds_protocol_delta>`{.interpreted-text
role="ref"}.)

To use delta, simply set the api_type field of your
`ApiConfigSource <envoy_v3_api_msg_config.core.v3.ApiConfigSource>`{.interpreted-text
role="ref"} proto(s) to DELTA_GRPC. That works for both xDS and ADS; for
ADS, it\'s the api_type field of
`DynamicResources.ads_config <envoy_v3_api_field_config.bootstrap.v3.Bootstrap.dynamic_resources>`{.interpreted-text
role="ref"}, as described in the previous section.
