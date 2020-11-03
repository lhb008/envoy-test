TLS Inspector {#config_listener_filters_tls_inspector}
=============

TLS Inspector listener filter allows detecting whether the transport
appears to be TLS or plaintext, and if it is TLS, it detects the [Server
Name Indication](https://en.wikipedia.org/wiki/Server_Name_Indication)
and/or [Application-Layer Protocol
Negotiation](https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation)
from the client. This can be used to select a
`FilterChain <envoy_v3_api_msg_config.listener.v3.FilterChain>`{.interpreted-text
role="ref"} via the
`server_names <envoy_v3_api_field_config.listener.v3.FilterChainMatch.server_names>`{.interpreted-text
role="ref"} and/or
`application_protocols <envoy_v3_api_field_config.listener.v3.FilterChainMatch.application_protocols>`{.interpreted-text
role="ref"} of a
`FilterChainMatch <envoy_v3_api_msg_config.listener.v3.FilterChainMatch>`{.interpreted-text
role="ref"}.

-   `SNI <faq_how_to_setup_sni>`{.interpreted-text role="ref"}
-   `v2 API reference <envoy_v3_api_field_config.listener.v3.ListenerFilter.name>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.listener.tls_inspector*.

Example
-------

A sample filter configuration could be:

``` {.yaml}
listener_filters:
- name: "envoy.filters.listener.tls_inspector"
  typed_config: {}
```

Statistics
----------

This filter has a statistics tree rooted at *tls_inspector* with the
following statistics:

  -------------------------------------------------------------------------------------------------------------------------------
  Name                     Type              Description
  ------------------------ ----------------- ------------------------------------------------------------------------------------
  connection_closed        Counter           Total connections closed

  client_hello_too_large   Counter           Total unreasonably large Client Hello received

  read_error               Counter           Total read errors

  tls_found                Counter           Total number of times TLS was found

  tls_not_found            Counter           Total number of times TLS was not found

  alpn_found               Counter           Total number of times [Application-Layer Protocol
                                             Negotiation](https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation)
                                             was successful

  alpn_not_found           Counter           Total number of times [Application-Layer Protocol
                                             Negotiation](https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation)
                                             has failed

  sni_found                Counter           Total number of times [Server Name
                                             Indication](https://en.wikipedia.org/wiki/Server_Name_Indication) was found

  sni_not_found            Counter           Total number of times [Server Name
                                             Indication](https://en.wikipedia.org/wiki/Server_Name_Indication) was not found
  -------------------------------------------------------------------------------------------------------------------------------
