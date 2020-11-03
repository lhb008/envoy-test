Local rate limiting {#arch_overview_local_rate_limit}
===================

Envoy supports local (non-distributed) rate limiting of L4 connections
via the
`local rate limit filter <config_network_filters_local_rate_limit>`{.interpreted-text
role="ref"}.

Envoy additionally supports local rate limiting of HTTP requests via the
`HTTP local rate limit filter <config_http_filters_local_rate_limit>`{.interpreted-text
role="ref"}. This can be activated globally at the listener level or at
a more specific level (e.g.: the virtual host or route level).

Finally, Envoy also supports
`global rate limiting <arch_overview_global_rate_limit>`{.interpreted-text
role="ref"}. Local rate limiting can be used in conjunction with global
rate limiting to reduce load on the global rate limit service.
