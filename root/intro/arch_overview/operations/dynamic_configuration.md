xDS configuration API overview {#arch_overview_dynamic_config}
==============================

Envoy is architected such that different types of configuration
management approaches are possible. The approach taken in a deployment
will be dependent on the needs of the implementor. Simple deployments
are possible with a fully static configuration. More complicated
deployments can incrementally add more complex dynamic configuration,
the downside being that the implementor must provide one or more
external gRPC/REST based configuration provider APIs. These APIs are
collectively known as `"xDS" <xds_protocol>`{.interpreted-text
role="ref"} (\* discovery service). This document gives an overview of
the options currently available.

-   Top level configuration `reference <config>`{.interpreted-text
    role="ref"}.
-   `Reference configurations <install_ref_configs>`{.interpreted-text
    role="ref"}.
-   Envoy `v3 API overview <config_overview>`{.interpreted-text
    role="ref"}.
-   `xDS API endpoints <config_overview_management_server>`{.interpreted-text
    role="ref"}.

Fully static
------------

In a fully static configuration, the implementor provides a set of
`listeners
<config_listeners>`{.interpreted-text role="ref"} (and
`filter chains <envoy_v3_api_msg_config.listener.v3.Filter>`{.interpreted-text
role="ref"}), `clusters
<config_cluster_manager>`{.interpreted-text role="ref"}, etc. Dynamic
host discovery is only possible via DNS based
`service discovery <arch_overview_service_discovery>`{.interpreted-text
role="ref"}. Configuration reloads must take place via the built in
`hot restart <arch_overview_hot_restart>`{.interpreted-text role="ref"}
mechanism.

Though simplistic, fairly complicated deployments can be created using
static configurations and graceful hot restarts.

EDS {#arch_overview_dynamic_config_eds}
---

The
`Endpoint Discovery Service (EDS) API <arch_overview_service_discovery_types_eds>`{.interpreted-text
role="ref"} provides a more advanced mechanism by which Envoy can
discover members of an upstream cluster. Layered on top of a static
configuration, EDS allows an Envoy deployment to circumvent the
limitations of DNS (maximum records in a response, etc.) as well as
consume more information used in load balancing and routing (e.g.,
canary status, zone, etc.).

CDS {#arch_overview_dynamic_config_cds}
---

The
`Cluster Discovery Service (CDS) API <config_cluster_manager_cds>`{.interpreted-text
role="ref"} layers on a mechanism by which Envoy can discover upstream
clusters used during routing. Envoy will gracefully add, update, and
remove clusters as specified by the API. This API allows implementors to
build a topology in which Envoy does not need to be aware of all
upstream clusters at initial configuration time. Typically, when doing
HTTP routing along with CDS (but without route discovery service),
implementors will make use of the router\'s ability to forward requests
to a cluster specified in an
`HTTP request header <envoy_v3_api_field_config.route.v3.RouteAction.cluster_header>`{.interpreted-text
role="ref"}.

Although it is possible to use CDS without EDS by specifying fully
static clusters, we recommend still using the EDS API for clusters
specified via CDS. Internally, when a cluster definition is updated, the
operation is graceful. However, all existing connection pools will be
drained and reconnected. EDS does not suffer from this limitation. When
hosts are added and removed via EDS, the existing hosts in the cluster
are unaffected.

RDS {#arch_overview_dynamic_config_rds}
---

The
`Route Discovery Service (RDS) API <config_http_conn_man_rds>`{.interpreted-text
role="ref"} layers on a mechanism by which Envoy can discover the entire
route configuration for an HTTP connection manager filter at runtime.
The route configuration will be gracefully swapped in without affecting
existing requests. This API, when used alongside EDS and CDS, allows
implementors to build a complex routing topology
(`traffic shifting <config_http_conn_man_route_table_traffic_splitting>`{.interpreted-text
role="ref"}, blue/green deployment, etc).

VHDS
----

The
`Virtual Host Discovery Service <config_http_conn_man_vhds>`{.interpreted-text
role="ref"} allows the virtual hosts belonging to a route configuration
to be requested as needed separately from the route configuration
itself. This API is typically used in deployments in which there are a
large number of virtual hosts in a route configuration.

SRDS
----

The
`Scoped Route Discovery Service (SRDS) API <arch_overview_http_routing_route_scope>`{.interpreted-text
role="ref"} allows a route table to be broken up into multiple pieces.
This API is typically used in deployments of HTTP routing with massive
route tables in which simple linear searches are not feasible.

LDS {#arch_overview_dynamic_config_lds}
---

The
`Listener Discovery Service (LDS) API <config_listeners_lds>`{.interpreted-text
role="ref"} layers on a mechanism by which Envoy can discover entire
listeners at runtime. This includes all filter stacks, up to and
including HTTP filters with embedded references to
`RDS <config_http_conn_man_rds>`{.interpreted-text role="ref"}. Adding
LDS into the mix allows almost every aspect of Envoy to be dynamically
configured. Hot restart should only be required for very rare
configuration changes (admin, tracing driver, etc.), certificate
rotation, or binary updates.

SDS
---

The
`Secret Discovery Service (SDS) API <config_secret_discovery_service>`{.interpreted-text
role="ref"} layers on a mechanism by which Envoy can discover
cryptographic secrets (certificate plus private key, TLS session ticket
keys) for its listeners, as well as configuration of peer certificate
validation logic (trusted root certs, revocations, etc).

RTDS
----

The
`RunTime Discovery Service (RTDS) API <config_runtime_rtds>`{.interpreted-text
role="ref"} allows `runtime <config_runtime>`{.interpreted-text
role="ref"} layers to be fetched via an xDS API. This may be favorable
to, or augmented by, file system layers.

ECDS
----

The
`Extension Config Discovery Service (ECDS) API <config_overview_extension_discovery>`{.interpreted-text
role="ref"} allows extension configurations (e.g. HTTP filter
configuration) to be served independently from the listener. This is
useful when building systems that are more appropriately split from the
primary control plane such as WAF, fault testing, etc.

Aggregated xDS (\"ADS\")
------------------------

EDS, CDS, etc. are each separate services, with different REST/gRPC
service names, e.g. StreamListeners, StreamSecrets. For users looking to
enforce the order in which resources of different types reach Envoy,
there is aggregated xDS, a single gRPC service that carries all resource
types in a single gRPC stream. (ADS is only supported by gRPC).
`More details about ADS <config_overview_ads>`{.interpreted-text
role="ref"}.

Delta gRPC xDS {#arch_overview_dynamic_config_delta}
--------------

Standard xDS is \"state-of-the-world\": every update must contain every
resource, with the absence of a resource from an update implying that
the resource is gone. Envoy supports a \"delta\" variant of xDS
(including ADS), where updates only contain resources
added/changed/removed. Delta xDS is a new protocol, with
request/response APIs different from SotW.
`More details about delta <config_overview_delta>`{.interpreted-text
role="ref"}.
