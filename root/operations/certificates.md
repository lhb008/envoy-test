Certificate Management {#operations_certificates}
======================

Envoy provides several mechanisms for cert management. At a high level
they can be broken into

1.  Static
    `CommonTlsContext <envoy_api_msg_auth.CommonTlsContext>`{.interpreted-text
    role="ref"} referenced certificates. These will *not* reload
    automatically, and requires either a restart of the proxy or
    reloading the clusters/listeners that reference them.
    `Hot restarting <arch_overview_hot_restart>`{.interpreted-text
    role="ref"} can be used here to pick up the new certificates without
    dropping traffic.
2.  `Secret Discovery Service <config_secret_discovery_service>`{.interpreted-text
    role="ref"} referenced certificates. By using SDS, certificates can
    either be referenced as files (reloading the certs when the parent
    directory is moved) or through an external SDS server that can push
    new certificates.
