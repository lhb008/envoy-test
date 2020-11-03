JSON Web Token (JWT) Authentication {#arch_overview_jwt_authn}
===================================

-   `HTTP filter configuration <config_http_filters_jwt_authn>`{.interpreted-text
    role="ref"}.

The JSON Web Token (JWT) Authentication filter checks if the incoming
request has a valid [JSON Web Token
(JWT)](https://tools.ietf.org/html/rfc7519). It checks the validity of
the JWT by verifying the JWT signature, audiences and issuer based on
the
`HTTP filter configuration <config_http_filters_jwt_authn>`{.interpreted-text
role="ref"}. The JWT Authentication filter could be configured to either
reject the request with invalid JWT immediately or defer the decision to
later filters by passing the JWT payload to other filters.

The JWT Authentication filter supports to check the JWT under various
conditions of the request, it could be configured to check JWT only on
specific paths so that you could allowlist some paths from the JWT
authentication, which is useful if a path is accessible publicly and
doesn\'t require any JWT authentication.

The JWT Authentication filter supports to extract the JWT from various
locations of the request and could combine multiple JWT requirements for
the same request. The [JSON Web Key Set
(JWKS)](https://tools.ietf.org/html/rfc7517) needed for the JWT
signature verification could be either specified inline in the filter
config or fetched from remote server via HTTP/HTTPS.

The JWT Authentication filter also supports to write the payloads of the
successfully verified JWT to
`Dynamic State <arch_overview_data_sharing_between_filters>`{.interpreted-text
role="ref"} so that later filters could use it to make their own
decisions based on the JWT payloads.
