Service to service plus front proxy {#deployment_type_front_proxy}
===================================

![image](/_static/front_proxy.svg)

The above diagram shows the
`service to service <deployment_type_service_to_service>`{.interpreted-text
role="ref"} configuration sitting behind an Envoy cluster used as an
HTTP L7 edge reverse proxy. The reverse proxy provides the following
features:

-   Terminates TLS.
-   Supports both HTTP/1.1 and HTTP/2.
-   Full HTTP L7 routing support.
-   Talks to the service to service Envoy clusters via the standard
    `ingress port
    <deployment_type_service_to_service_ingress>`{.interpreted-text
    role="ref"} and using the discovery service for host lookup. Thus,
    the front Envoy hosts work identically to any other Envoy host,
    other than the fact that they do not run collocated with another
    service. This means that are operated in the same way and emit the
    same statistics.

Configuration template
----------------------

The source distribution includes an example front proxy configuration
that is very similar to the version that Lyft runs in production. See
`here <install_ref_configs>`{.interpreted-text role="ref"} for more
information.
