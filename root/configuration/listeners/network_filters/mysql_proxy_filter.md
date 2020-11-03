MySQL proxy {#config_network_filters_mysql_proxy}
===========

The MySQL proxy filter decodes the wire protocol between the MySQL
client and server. It decodes the SQL queries in the payload (SQL99
format only). The decoded info is emitted as dynamic metadata that can
be combined with access log filters to get detailed information on
tables accessed as well as operations performed on each table.

::: {.attention}
::: {.title}
Attention
:::

The mysql_proxy filter is experimental and is currently under active
development. Capabilities will be expanded over time and the
configuration structures are likely to change.
:::

::: {.warning}
::: {.title}
Warning
:::

The mysql_proxy filter was tested with MySQL v5.5. The filter may not
work with other versions of MySQL due to differences in the protocol
implementation.
:::

Configuration {#config_network_filters_mysql_proxy_config}
-------------

The MySQL proxy filter should be chained with the TCP proxy filter as
shown in the configuration snippet below:

``` {.yaml}
filter_chains:
- filters:
  - name: envoy.filters.network.mysql_proxy
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.filters.network.mysql_proxy.v3.MySQLProxy
      stat_prefix: mysql
  - name: envoy.filters.network.tcp_proxy
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.filters.network.tcp_proxy.v3.TcpProxy
      stat_prefix: tcp
      cluster: ...
```

Statistics {#config_network_filters_mysql_proxy_stats}
----------

Every configured MySQL proxy filter has statistics rooted at
*mysql.\<stat_prefix\>.* with the following statistics:

  ---------------------------------------------------------------------------
  Name                  Type              Description
  --------------------- ----------------- -----------------------------------
  auth_switch_request   Counter           Number of times the upstream server
                                          requested clients to switch to a
                                          different authentication method

  decoder_errors        Counter           Number of MySQL protocol decoding
                                          errors

  login_attempts        Counter           Number of login attempts

  login_failures        Counter           Number of login failures

  protocol_errors       Counter           Number of out of sequence protocol
                                          messages encountered in a session

  queries_parse_error   Counter           Number of MySQL queries parsed with
                                          errors

  queries_parsed        Counter           Number of MySQL queries
                                          successfully parsed

  sessions              Counter           Number of MySQL sessions since
                                          start

  upgraded_to_ssl       Counter           Number of sessions/connections that
                                          were upgraded to SSL
  ---------------------------------------------------------------------------

Dynamic Metadata {#config_network_filters_mysql_proxy_dynamic_metadata}
----------------

The MySQL filter emits the following dynamic metadata for each SQL query
parsed:

  -----------------------------------------------------------------------------------------
  Name              Type              Description
  ----------------- ----------------- -----------------------------------------------------
  \<table.db\>      string            The resource name in *table.db* format. The resource
                                      name defaults to the table being accessed if the
                                      database cannot be inferred.

  \[\]              list              A list of strings representing the operations
                                      executed on the resource. Operations can be one of
                                      insert/update/select/drop/delete/create/alter/show.
  -----------------------------------------------------------------------------------------

RBAC Enforcement on Table Accesses {#config_network_filters_mysql_proxy_rbac}
----------------------------------

The dynamic metadata emitted by the MySQL filter can be used in
conjunction with the RBAC filter to control accesses to individual
tables in a database. The following configuration snippet shows an
example RBAC filter configuration that denies SQL queries with
\_[update]() statements to the \_[catalog]() table in the
\_[productdb]() database.

``` {.yaml}
filter_chains:
- filters:
  - name: envoy.filters.network.mysql_proxy
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.filters.network.mysql_proxy.v3.MySQLProxy
      stat_prefix: mysql
  - name: envoy.filters.network.rbac
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.filters.network.rbac.v3.RBAC
      stat_prefix: rbac
      rules:
        action: DENY
        policies:
          "product-viewer":
            permissions:
            - metadata:
                filter: envoy.filters.network.mysql_proxy
                path:
                - key: catalog.productdb
                value:
                  list_match:
                    one_of:
                      string_match:
                        exact: update
            principals:
            - any: true
  - name: envoy.filters.network.tcp_proxy
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.filters.network.tcp_proxy.v3.TcpProxy
      stat_prefix: tcp
      cluster: mysql
```
