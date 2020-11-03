Reference configurations {#install_ref_configs}
========================

The source distribution includes a set of example configuration
templates for each of the three major Envoy deployment types:

-   `Service to service <deployment_type_service_to_service>`{.interpreted-text
    role="ref"}
-   `Front proxy <deployment_type_front_proxy>`{.interpreted-text
    role="ref"}
-   `Double proxy <deployment_type_double_proxy>`{.interpreted-text
    role="ref"}

The goal of this set of example configurations is to demonstrate the
full capabilities of Envoy in a complex deployment. All features will
not be applicable to all use cases. For full documentation see the
`configuration reference <config>`{.interpreted-text role="ref"}.

Configuration generator
-----------------------

Envoy configurations can become relatively complicated. At Lyft we use
[jinja](http://jinja.pocoo.org/) templating to make the configurations
easier to create and manage. The source distribution includes a version
of the configuration generator that loosely approximates what we use at
Lyft. We have also included three example configuration templates for
each of the above three scenarios.

-   Generator script: `configs/configgen.py`{.interpreted-text
    role="repo"}
-   Service to service template:
    `configs/envoy_service_to_service_v2.template.yaml`{.interpreted-text
    role="repo"}
-   Front proxy template:
    `configs/envoy_front_proxy_v2.template.yaml`{.interpreted-text
    role="repo"}
-   Double proxy template:
    `configs/envoy_double_proxy_v2.template.yaml`{.interpreted-text
    role="repo"}

To generate the example configurations run the following from the root
of the repo:

``` {.console}
mkdir -p generated/configs
bazel build //configs:example_configs
tar xvf $PWD/bazel-out/k8-fastbuild/bin/configs/example_configs.tar -C generated/configs
```

The previous command will produce three fully expanded configurations
using some variables defined inside of [configgen.py]{.title-ref}. See
the comments inside of [configgen.py]{.title-ref} for detailed
information on how the different expansions work.

A few notes about the example configurations:

-   An instance of
    `endpoint discovery service <arch_overview_service_discovery_types_eds>`{.interpreted-text
    role="ref"} is assumed to be running at
    [discovery.yourcompany.net]{.title-ref}.
-   DNS for [yourcompany.net]{.title-ref} is assumed to be setup for
    various things. Search the configuration templates for different
    instances of this.
-   Tracing is configured for [LightStep](https://lightstep.com/). To
    disable this or enable [Zipkin](https://zipkin.io) or
    [Datadog](https://datadoghq.com) tracing, delete or change the
    `tracing configuration <envoy_api_file_envoy/config/trace/v2/trace.proto>`{.interpreted-text
    role="ref"} accordingly.
-   The configuration demonstrates the use of a
    `global rate limiting service
    <arch_overview_global_rate_limit>`{.interpreted-text role="ref"}. To
    disable this delete the `rate limit configuration
    <config_rate_limit_service>`{.interpreted-text role="ref"}.
-   `Route discovery service <config_http_conn_man_rds>`{.interpreted-text
    role="ref"} is configured for the service to service reference
    configuration and it is assumed to be running at
    [rds.yourcompany.net]{.title-ref}.
-   `Cluster discovery service <config_cluster_manager_cds>`{.interpreted-text
    role="ref"} is configured for the service to service reference
    configuration and it is assumed that be running at
    [cds.yourcompany.net]{.title-ref}.
