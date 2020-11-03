Initialization {#arch_overview_initialization}
==============

How Envoy initializes itself when it starts up is complex. This section
explains at a high level how the process works. All of the following
happens before any listeners start listening and accepting new
connections.

-   During startup, the
    `cluster manager <arch_overview_cluster_manager>`{.interpreted-text
    role="ref"} goes through a multi-phase initialization where it first
    initializes static/DNS clusters, then predefined
    `EDS <arch_overview_dynamic_config_eds>`{.interpreted-text
    role="ref"} clusters. Then it initializes
    `CDS <arch_overview_dynamic_config_cds>`{.interpreted-text
    role="ref"} if applicable, waits for one response (or failure) for a
    `bounded period of time <envoy_v3_api_field_config.core.v3.ConfigSource.initial_fetch_timeout>`{.interpreted-text
    role="ref"}, and does the same primary/secondary initialization of
    CDS provided clusters.
-   If clusters use
    `active health checking <arch_overview_health_checking>`{.interpreted-text
    role="ref"}, Envoy also does a single active health check round.
-   Once cluster manager initialization is done,
    `RDS <arch_overview_dynamic_config_rds>`{.interpreted-text
    role="ref"} and
    `LDS <arch_overview_dynamic_config_lds>`{.interpreted-text
    role="ref"} initialize (if applicable). The server waits for a
    `bounded period of time <envoy_v3_api_field_config.core.v3.ConfigSource.initial_fetch_timeout>`{.interpreted-text
    role="ref"} for at least one response (or failure) for LDS/RDS
    requests. After which, it starts accepting connections.
-   If LDS itself returns a listener that needs an RDS response, Envoy
    further waits for a
    `bounded period of time <envoy_v3_api_field_config.core.v3.ConfigSource.initial_fetch_timeout>`{.interpreted-text
    role="ref"} until an RDS response (or failure) is received. Note
    that this process takes place on every future listener addition via
    LDS and is known as
    `listener warming <config_listeners_lds>`{.interpreted-text
    role="ref"}.
-   After all of the previous steps have taken place, the listeners
    start accepting new connections. This flow ensures that during hot
    restart the new process is fully capable of accepting and processing
    new connections before the draining of the old process begins.

A key design principle of initialization is that an Envoy is always
guaranteed to initialize within
`initial_fetch_timeout <envoy_v3_api_field_config.core.v3.ConfigSource.initial_fetch_timeout>`{.interpreted-text
role="ref"}, with a best effort made to obtain the complete set of xDS
configuration within that subject to the management server availability.
