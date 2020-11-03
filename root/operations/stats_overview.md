Statistics overview {#operations_stats}
===================

Envoy outputs numerous statistics which depend on how the server is
configured. They can be seen locally via the
:<http:get>:[/stats]{.title-ref} command and are typically sent to a
`statsd cluster
<arch_overview_statistics>`{.interpreted-text role="ref"}. The
statistics that are output are documented in the relevant sections of
the `configuration guide <config>`{.interpreted-text role="ref"}. Some
of the more important statistics that will almost always be used can be
found in the following sections:

-   `HTTP connection manager <config_http_conn_man_stats>`{.interpreted-text
    role="ref"}
-   `Upstream cluster <config_cluster_manager_cluster_stats>`{.interpreted-text
    role="ref"}
