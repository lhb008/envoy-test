Rate limit service {#config_rate_limit_service}
==================

The
`rate limit service <arch_overview_global_rate_limit>`{.interpreted-text
role="ref"} configuration specifies the global rate limit service Envoy
should talk to when it needs to make global rate limit decisions. If no
rate limit service is configured, a \"null\" service will be used which
will always return OK if called.

-   `v3 API reference <envoy_v3_api_msg_config.ratelimit.v3.RateLimitServiceConfig>`{.interpreted-text
    role="ref"}

gRPC service IDL
----------------

Envoy expects the rate limit service to support the gRPC IDL specified
in
`rls.proto <envoy_v3_api_file_envoy/service/ratelimit/v3/rls.proto>`{.interpreted-text
role="ref"}. See the IDL documentation for more information on how the
API works. See Lyft\'s reference implementation
[here](https://github.com/lyft/ratelimit).
