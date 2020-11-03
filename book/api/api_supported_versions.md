Supported API versions {#api_supported_versions}
======================

Envoy\'s APIs follow a
`versioning scheme <api/API_VERSIONING.md>`{.interpreted-text
role="repo"} in which Envoy supports multiple major API versions at any
point in time. The following versions are currently supported:

-   `v2 xDS API <envoy_api_reference>`{.interpreted-text role="ref"}
    (*deprecated*, end-of-life EOY 2020). This API will not accept new
    features after the end of Q1 2020.
-   `v3 xDS API <envoy_v3_api_reference>`{.interpreted-text role="ref"}
    (*active*, end-of-life unknown). Envoy developers and operators are
    encouraged to be actively adopting and working with v3 xDS.

The following API versions are no longer supported by Envoy:

-   v1 xDS API. This was the legacy REST-JSON API that preceded the
    current Protobuf and dual REST/gRPC xDS APIs.
