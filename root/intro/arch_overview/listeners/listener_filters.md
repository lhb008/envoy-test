Listener filters {#arch_overview_listener_filters}
================

As discussed in the
`listener <arch_overview_listeners>`{.interpreted-text role="ref"}
section, listener filters may be used to manipulate connection metadata.
The main purpose of listener filters is to make adding further system
integration functions easier by not requiring changes to Envoy core
functionality, and also make interaction between multiple such features
more explicit.

The API for listener filters is relatively simple since ultimately these
filters operate on newly accepted sockets. Filters in the chain can stop
and subsequently continue iteration to further filters. This allows for
more complex scenarios such as calling a `rate limiting
service <arch_overview_global_rate_limit>`{.interpreted-text
role="ref"}, etc. Envoy already includes several listener filters that
are documented in this architecture overview as well as the
`configuration reference
<config_listener_filters>`{.interpreted-text role="ref"}.
