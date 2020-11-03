What is the status of incremental xDS support?
==============================================

The `incremental xDS <xds_protocol_delta>`{.interpreted-text role="ref"}
protocol is designed to improve efficiency, scalability and functional
use of xDS updates via two mechanisms:

-   Delta xDS. Resource deltas are delivered rather than
    state-of-the-world.
-   On-demand xDS. Resource can be lazy loaded depending on request
    contents.

Currently, all xDS protocols (including ADS) support delta xDS.
On-demand xDS is supported for
`VHDS <config_http_conn_man_vhds>`{.interpreted-text role="ref"} only.
