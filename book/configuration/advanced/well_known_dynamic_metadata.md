Well Known Dynamic Metadata {#well_known_dynamic_metadata}
===========================

Filters can emit dynamic metadata via the *setDynamicMetadata* routine
in the
`StreamInfo <include/envoy/stream_info/stream_info.h>`{.interpreted-text
role="repo"} interface on a
`Connection <include/envoy/network/connection.h>`{.interpreted-text
role="repo"}. This metadata emitted by a filter can be consumed by other
filters and useful features can be built by stacking such filters. For
example, a logging filter can consume dynamic metadata from an RBAC
filter to log details about runtime shadow rule behavior. Another
example is where an RBAC filter permits/restricts MySQL/MongoDB
operations by looking at the operational metadata emitted by the MongoDB
filter.

The following Envoy filters emit dynamic metadata that other filters can
leverage.

-   `External Authorization Filter <config_http_filters_ext_authz_dynamic_metadata>`{.interpreted-text
    role="ref"}
-   `External Authorization Network Filter <config_network_filters_ext_authz_dynamic_metadata>`{.interpreted-text
    role="ref"}
-   `Mongo Proxy Filter <config_network_filters_mongo_proxy_dynamic_metadata>`{.interpreted-text
    role="ref"}
-   `MySQL Proxy Filter <config_network_filters_mysql_proxy_dynamic_metadata>`{.interpreted-text
    role="ref"}
-   `Postgres Proxy Filter <config_network_filters_postgres_proxy_dynamic_metadata>`{.interpreted-text
    role="ref"}
-   `Role Based Access Control (RBAC) Filter <config_http_filters_rbac_dynamic_metadata>`{.interpreted-text
    role="ref"}
-   `Role Based Access Control (RBAC) Network Filter <config_network_filters_rbac_dynamic_metadata>`{.interpreted-text
    role="ref"}
-   `ZooKeeper Proxy Filter <config_network_filters_zookeeper_proxy_dynamic_metadata>`{.interpreted-text
    role="ref"}

The following Envoy filters can be configured to consume dynamic
metadata emitted by other filters.

-   `External Authorization Filter via the metadata context namespaces
    <envoy_v3_api_field_extensions.filters.http.ext_authz.v3.ExtAuthz.metadata_context_namespaces>`{.interpreted-text
    role="ref"}
-   `RateLimit Filter limit override <config_http_filters_rate_limit_override_dynamic_metadata>`{.interpreted-text
    role="ref"}

Shared Dynamic Metadata {#shared_dynamic_metadata}
-----------------------

Dynamic metadata that is set by multiple filters is placed in the common
key namespace [envoy.common]{.title-ref}. Refer to the corresponding
rules when setting this metadata.

  ---------------------------------------------------------------------------------
  Name              Type      Description                Rules
  ----------------- --------- -------------------------- --------------------------
  access_log_hint   boolean   Whether access loggers     When this metadata is
                              should log the request.    already set: A
                                                         [true]{.title-ref} value
                                                         should not be overwritten
                                                         by a [false]{.title-ref}
                                                         value, while a
                                                         [false]{.title-ref} value
                                                         can be overwritten by a
                                                         [true]{.title-ref} value.

  ---------------------------------------------------------------------------------

The following Envoy filters emit shared dynamic metadata.

-   `Role Based Access Control (RBAC) Filter <config_http_filters_rbac_dynamic_metadata>`{.interpreted-text
    role="ref"}
-   `Role Based Access Control (RBAC) Network Filter <config_network_filters_rbac_dynamic_metadata>`{.interpreted-text
    role="ref"}

The following filters consume shared dynamic metadata.

-   `Metadata Access Log Filter<envoy_v3_api_msg_config.accesslog.v3.MetadataFilter>`{.interpreted-text
    role="ref"}
