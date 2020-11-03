How do I configure SNI for listeners? {#faq_how_to_setup_sni}
=====================================

[SNI](https://en.wikipedia.org/wiki/Server_Name_Indication) is only
supported in the `v3
configuration/API <config_overview>`{.interpreted-text role="ref"}.

::: {.attention}
::: {.title}
Attention
:::

`TLS Inspector <config_listener_filters_tls_inspector>`{.interpreted-text
role="ref"} listener filter must be configured in order to detect
requested SNI.
:::

The following is a YAML example of the above requirement.

``` {.yaml}
address:
  socket_address: { address: 127.0.0.1, port_value: 1234 }
listener_filters:
- name: "envoy.filters.listener.tls_inspector"
  typed_config: {}
filter_chains:
- filter_chain_match:
    server_names: ["example.com", "www.example.com"]
  transport_socket:
    name: envoy.transport_sockets.tls
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.DownstreamTlsContext
      common_tls_context:
        tls_certificates:
        - certificate_chain: { filename: "example_com_cert.pem" }
          private_key: { filename: "example_com_key.pem" }
  filters:
  - name: envoy.filters.network.http_connection_manager
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
      stat_prefix: ingress_http
      route_config:
        virtual_hosts:
        - name: default
          domains: "*"
          routes:
          - match: { prefix: "/" }
            route: { cluster: service_foo }
- filter_chain_match:
    server_names: "api.example.com"
  transport_socket:
    name: envoy.transport_sockets.tls
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.DownstreamTlsContext
      common_tls_context:
        tls_certificates:
        - certificate_chain: { filename: "api_example_com_cert.pem" }
          private_key: { filename: "api_example_com_key.pem" }
  filters:
  - name: envoy.filters.network.http_connection_manager
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
      stat_prefix: ingress_http
      route_config:
        virtual_hosts:
        - name: default
          domains: "*"
          routes:
          - match: { prefix: "/" }
            route: { cluster: service_foo }
```

How do I configure SNI for clusters?
====================================

For clusters, a fixed SNI can be set in
`UpstreamTlsContext <envoy_v3_api_field_extensions.transport_sockets.tls.v3.UpstreamTlsContext.sni>`{.interpreted-text
role="ref"}. To derive SNI from HTTP [host]{.title-ref} or
[:authority]{.title-ref} header, turn on
`auto_sni <envoy_v3_api_field_config.core.v3.UpstreamHttpProtocolOptions.auto_sni>`{.interpreted-text
role="ref"} to override the fixed SNI in
[UpstreamTlsContext]{.title-ref}. If upstream will present certificates
with the hostname in SAN, turn on
`auto_san_validation <envoy_v3_api_field_config.core.v3.UpstreamHttpProtocolOptions.auto_san_validation>`{.interpreted-text
role="ref"} too. It still needs a trust CA in validation context in
[UpstreamTlsContext]{.title-ref} for trust anchor.
