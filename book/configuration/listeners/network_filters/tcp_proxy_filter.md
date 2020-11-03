TCP proxy {#config_network_filters_tcp_proxy}
=========

-   TCP proxy
    `architecture overview <arch_overview_tcp_proxy>`{.interpreted-text
    role="ref"}
-   `v3 API reference <envoy_v3_api_msg_extensions.filters.network.tcp_proxy.v3.TcpProxy>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.network.tcp_proxy*.

Dynamic cluster selection {#config_network_filters_tcp_proxy_dynamic_cluster}
-------------------------

The upstream cluster used by the TCP proxy filter can be dynamically set
by other network filters on a per-connection basis by setting a
per-connection state object under the key
[envoy.tcp_proxy.cluster]{.title-ref}. See the implementation for the
details.

Routing to a subset of hosts {#config_network_filters_tcp_proxy_subset_lb}
----------------------------

TCP proxy can be configured to route to a subset of hosts within an
upstream cluster.

To define metadata that a suitable upstream host must match, use one of
the following fields:

1.  Use
    `TcpProxy.metadata_match<envoy_v3_api_field_extensions.filters.network.tcp_proxy.v3.TcpProxy.metadata_match>`{.interpreted-text
    role="ref"} to define required metadata for a single upstream
    cluster.
2.  Use
    `ClusterWeight.metadata_match<envoy_v3_api_field_extensions.filters.network.tcp_proxy.v3.TcpProxy.WeightedCluster.ClusterWeight.metadata_match>`{.interpreted-text
    role="ref"} to define required metadata for a weighted upstream
    cluster.
3.  Use combination of
    `TcpProxy.metadata_match<envoy_v3_api_field_extensions.filters.network.tcp_proxy.v3.TcpProxy.metadata_match>`{.interpreted-text
    role="ref"} and
    `ClusterWeight.metadata_match<envoy_v3_api_field_extensions.filters.network.tcp_proxy.v3.TcpProxy.WeightedCluster.ClusterWeight.metadata_match>`{.interpreted-text
    role="ref"} to define required metadata for a weighted upstream
    cluster (metadata from the latter will be merged on top of the
    former).

In addition, dynamic metadata can be set by earlier network filters on
the [StreamInfo]{.title-ref}. Setting the dynamic metadata must happen
before [onNewConnection()]{.title-ref} is called on the
[TcpProxy]{.title-ref} filter to affect load balancing.

Statistics {#config_network_filters_tcp_proxy_stats}
----------

The TCP proxy filter emits both its own downstream statistics as well as
many of the `cluster
upstream statistics <config_cluster_manager_cluster_stats>`{.interpreted-text
role="ref"} where applicable. The downstream statistics are rooted at
*tcp.\<stat_prefix\>.* with the following statistics:

  ------------------------------------------------------------------------------------------------------
  Name                                            Type              Description
  ----------------------------------------------- ----------------- ------------------------------------
  downstream_cx_total                             Counter           Total number of connections handled
                                                                    by the filter

  downstream_cx_no_route                          Counter           Number of connections for which no
                                                                    matching route was found or the
                                                                    cluster for the route was not found

  downstream_cx_tx_bytes_total                    Counter           Total bytes written to the
                                                                    downstream connection

  downstream_cx_tx_bytes_buffered                 Gauge             Total bytes currently buffered to
                                                                    the downstream connection

  downstream_cx_rx_bytes_total                    Counter           Total bytes read from the downstream
                                                                    connection

  downstream_cx_rx_bytes_buffered                 Gauge             Total bytes currently buffered from
                                                                    the downstream connection

  downstream_flow_control_paused_reading_total    Counter           Total number of times flow control
                                                                    paused reading from downstream

  downstream_flow_control_resumed_reading_total   Counter           Total number of times flow control
                                                                    resumed reading from downstream

  idle_timeout                                    Counter           Total number of connections closed
                                                                    due to idle timeout

  max_downstream_connection_duration              Counter           Total number of connections closed
                                                                    due to
                                                                    max_downstream_connection_duration
                                                                    timeout

  upstream_flush_total                            Counter           Total number of connections that
                                                                    continued to flush upstream data
                                                                    after the downstream connection was
                                                                    closed

  upstream_flush_active                           Gauge             Total connections currently
                                                                    continuing to flush upstream data
                                                                    after the downstream connection was
                                                                    closed
  ------------------------------------------------------------------------------------------------------
