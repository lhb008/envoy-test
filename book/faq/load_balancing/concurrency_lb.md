Why doesn\'t RR load balancing appear to be even?
=================================================

Envoy utilizes a siloed
`threading model <arch_overview_threading>`{.interpreted-text
role="ref"}. This means that worker threads and the load balancers that
run on them do not coordinate with each other. When utilizing load
balancing policies such as
`round robin <arch_overview_load_balancing_types_round_robin>`{.interpreted-text
role="ref"}, it may thus appear that load balancing is not working
properly when using multiple workers. The
`--concurrency`{.interpreted-text role="option"} option can be used to
adjust the number of workers if desired.

The siloed execution model is also the reason why multiple HTTP/2
connections may be established to each upstream;
`connection pools <arch_overview_conn_pool>`{.interpreted-text
role="ref"} are not shared between workers.
