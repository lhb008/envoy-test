Client TLS authentication {#config_network_filters_client_ssl_auth}
=========================

-   Client TLS authentication filter
    `architecture overview <arch_overview_ssl_auth_filter>`{.interpreted-text
    role="ref"}
-   `v3 API reference <envoy_v3_api_msg_extensions.filters.network.client_ssl_auth.v3.ClientSSLAuth>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.network.client_ssl_auth*.

Statistics {#config_network_filters_client_ssl_auth_stats}
----------

Every configured client TLS authentication filter has statistics rooted
at *auth.clientssl.\<stat_prefix\>.* with the following statistics:

  ----------------------------------------------------------------------------
  Name                   Type              Description
  ---------------------- ----------------- -----------------------------------
  update_success         Counter           Total principal update successes

  update_failure         Counter           Total principal update failures

  auth_no_ssl            Counter           Total connections ignored due to no
                                           TLS

  auth_ip_allowlist      Counter           Total connections allowed due to
                                           the IP allowlist

  auth_digest_match      Counter           Total connections allowed due to
                                           certificate match

  auth_digest_no_match   Counter           Total connections denied due to no
                                           certificate match

  total_principals       Gauge             Total loaded principals
  ----------------------------------------------------------------------------

REST API {#config_network_filters_client_ssl_auth_rest_api}
--------
