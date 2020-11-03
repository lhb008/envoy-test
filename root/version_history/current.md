1.16.0 (October 8, 2020)
========================

Incompatible Behavior Changes
-----------------------------

*Changes that are expected to cause an incompatibility if applicable;
deployment changes are likely required*

-   build: added visibility rules for upstream. If these cause
    visibility related breakage, see notes in
    `BUILD <BUILD>`{.interpreted-text role="repo"}.
-   build: tcmalloc changes require Clang 9. This requirement change can
    be avoided by building with [\--define
    tcmalloc=gperftools]{.title-ref} to use the older tcmalloc code.
-   config: additional warnings have been added for the use of v2 APIs.
    These appear as log messages and are also captured in the
    `deprecated_feature_use <runtime_stats>`{.interpreted-text
    role="ref"} counter after server initialization.
-   dns:
    [envoy.restart_features.use_apple_api_for_dns_lookups]{.title-ref}
    is on by default. This flag only affects Apple platforms (macOS,
    iOS). It is incompatible to have the runtime flag set to true at the
    same time as specifying the `use_tcp_for_dns_lookups` option or
    custom dns resolvers. Doing so will cause failure.
-   watchdog: added two guarddogs, breaking the aggregated stats for the
    single guarddog system. The aggregated stats for the guarddogs will
    have the following prefixes: [main_thread]{.title-ref} and
    [workers]{.title-ref}. Concretely, anything monitoring
    [server.watchdog_miss]{.title-ref} and
    [server.watchdog_mega_miss]{.title-ref} will need to be updated.

Minor Behavior Changes
----------------------

*Changes that may cause incompatibilities for some users, but should not
for most*

-   adaptive concurrency: added a response body / grpc-message header
    for rejected requests.
-   async_client: minor change to handling header only responses more
    similar to header-with-empty-body responses.
-   build: an
    `Ubuntu based debug image <install_binaries>`{.interpreted-text
    role="ref"} is built and published in DockerHub.
-   build: the debug information will be generated separately to reduce
    target size and reduce compilation time when build in compilation
    mode [dbg]{.title-ref} and [opt]{.title-ref}. Users will need to
    build dwp file to debug with gdb.
-   compressor: always insert [Vary]{.title-ref} headers for
    compressible resources even if it\'s decided not to compress a
    response due to incompatible [Accept-Encoding]{.title-ref} value.
    The [Vary]{.title-ref} header needs to be inserted to let a caching
    proxy in front of Envoy know that the requested resource still can
    be served with compression applied.
-   decompressor: headers-only requests were incorrectly not advertising
    accept-encoding when configured to do so. This is now fixed.
-   ext_authz filter: request timeout will now count from the time the
    check request is created, instead of when it becomes active. This
    makes sure that the timeout is enforced even if the ext_authz
    cluster\'s circuit breaker is engaged. This behavior can be reverted
    by setting runtime feature
    [envoy.reloadable_features.ext_authz_measure_timeout_on_check_created]{.title-ref}
    to false. When enabled, a new [ext_authz.timeout]{.title-ref} stat
    is counted when timeout occurs. See
    `stats <config_http_filters_ext_authz_stats>`{.interpreted-text
    role="ref"}.
-   grpc reverse bridge: upstream headers will no longer be propagated
    when the response is missing or contains an unexpected content-type.
-   http: added
    `contains <envoy_api_msg_type.matcher.StringMatcher>`{.interpreted-text
    role="ref"}, a new string matcher type which matches if the value of
    the string has the substring mentioned in contains matcher.
-   http: added
    `contains <envoy_api_msg_route.HeaderMatcher>`{.interpreted-text
    role="ref"}, a new header matcher type which matches if the value of
    the header has the substring mentioned in contains matcher.
-   http: added
    `headers_to_add <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.ResponseMapper.headers_to_add>`{.interpreted-text
    role="ref"} to
    `local reply mapper <config_http_conn_man_local_reply>`{.interpreted-text
    role="ref"} to allow its users to add/append/override response HTTP
    headers to local replies.
-   http: added HCM level configuration of
    `error handling on invalid messaging <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stream_error_on_invalid_http_message>`{.interpreted-text
    role="ref"} which substantially changes Envoy\'s behavior when
    encountering invalid HTTP/1.1 defaulting to closing the connection
    instead of allowing reuse. This can temporarily be reverted by
    setting
    [envoy.reloadable_features.hcm_stream_error_on_invalid_message]{.title-ref}
    to false, or permanently reverted by setting the HTTP/1
    configuration
    `override_stream_error_on_invalid_http_message <envoy_v3_api_field_config.core.v3.Http1ProtocolOptions.override_stream_error_on_invalid_http_message>`{.interpreted-text
    role="ref"} to true to restore prior HTTP/1.1 behavior (i.e.
    connection isn\'t terminated) and to retain prior HTTP/2 behavior
    (i.e. connection is terminated).
-   http: added HCM level configuration of
    `error handling on invalid messaging <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stream_error_on_invalid_http_message>`{.interpreted-text
    role="ref"} which substantially changes Envoy\'s behavior when
    encountering invalid HTTP/1.1 defaulting to closing the connection
    instead of allowing reuse. This can temporarily be reverted by
    setting
    [envoy.reloadable_features.hcm_stream_error_on_invalid_message]{.title-ref}
    to false, or permanently reverted by setting the
    `HCM option <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stream_error_on_invalid_http_message>`{.interpreted-text
    role="ref"} to true to restore prior HTTP/1.1 beavior and setting
    the *new* HTTP/2 configuration
    `override_stream_error_on_invalid_http_message <envoy_v3_api_field_config.core.v3.Http2ProtocolOptions.override_stream_error_on_invalid_http_message>`{.interpreted-text
    role="ref"} to false to retain prior HTTP/2 behavior.
-   http: applying route level header modifications to local replies
    sent on that route. This behavior may be temporarily reverted by
    setting
    [envoy.reloadable_features.always_apply_route_header_rules]{.title-ref}
    to false.
-   http: changed Envoy to send GOAWAY to HTTP2 downstreams when the
    `disable_keepalive <config_overload_manager_overload_actions>`{.interpreted-text
    role="ref"} overload action is active. This behavior may be
    temporarily reverted by setting
    [envoy.reloadable_features.overload_manager_disable_keepalive_drain_http2]{.title-ref}
    to false.
-   http: changed Envoy to send error headers and body when possible.
    This behavior may be temporarily reverted by setting
    [envoy.reloadable_features.allow_response_for_timeout]{.title-ref}
    to false.
-   http: changed empty trailers encoding behavior by sending empty data
    with `end_stream` true (instead of sending empty trailers) for
    HTTP/2. This behavior can be reverted temporarily by setting runtime
    feature
    [envoy.reloadable_features.http2_skip_encoding_empty_trailers]{.title-ref}
    to false.
-   http: changed how local replies are processed for requests which
    transform from grpc to not-grpc, or not-grpc to grpc. Previously the
    initial generated reply depended on which filter sent the reply, but
    now the reply is consistently generated the way the downstream
    expects. This behavior can be temporarily reverted by setting
    [envoy.reloadable_features.unify_grpc_handling]{.title-ref} to
    false.
-   http: clarified and enforced 1xx handling. Multiple 100-continue
    headers are coalesced when proxying. 1xx headers other than {100,
    101} are dropped.
-   http: fixed a bug in access logs where early stream termination
    could be incorrectly tagged as a downstream disconnect, and
    disconnects after partial response were not flagged.
-   http: fixed the 100-continue response path to properly handle
    upstream failure by sending 5xx responses. This behavior can be
    temporarily reverted by setting
    [envoy.reloadable_features.allow_500_after_100]{.title-ref} to
    false.
-   http: the per-stream FilterState maintained by the HTTP connection
    manager will now provide read/write access to the downstream
    connection FilterState. As such, code that relies on interacting
    with this might see a change in behavior.
-   logging: added fine-grain logging for file level log control with
    logger management at administration interface. It can be enabled by
    option `--enable-fine-grain-logging`{.interpreted-text
    role="option"}.
-   logging: changed default log format to [\"\[%Y-%m-%d
    %T.%e\]\[%t\]\[%l\]\[%n\] \[%g:%\#\] %v\"]{.title-ref} and default
    value of `--log-format-prefix-with-location`{.interpreted-text
    role="option"} to [0]{.title-ref}.
-   logging: nghttp2 log messages no longer appear at trace level unless
    [ENVOY_NGHTTP2_TRACE]{.title-ref} is set in the environment.
-   lua: changed the response body returned by [httpCall()]{.title-ref}
    API to raw data. Previously, the returned data was string.
-   memory: switched to the [new
    tcmalloc](https://github.com/google/tcmalloc) for linux_x86_64
    builds. The [old tcmalloc](https://github.com/gperftools/gperftools)
    can still be enabled with the [\--define
    tcmalloc=gperftools]{.title-ref} option.
-   postgres: changed log format to tokenize fields of Postgres
    messages.
-   router: added transport failure reason to response body when
    upstream reset happens. After this change, the response body will be
    of the form [upstream connect error or disconnect/reset before
    headers. reset reason:{}, transport failure
    reason:{}]{.title-ref}.This behavior may be reverted by setting
    runtime feature
    [envoy.reloadable_features.http_transport_failure_reason_in_body]{.title-ref}
    to false.
-   router: now consumes all retry related headers to prevent them from
    being propagated to the upstream. This behavior may be reverted by
    setting runtime feature
    [envoy.reloadable_features.consume_all_retry_headers]{.title-ref} to
    false.
-   stats: the fake symbol table implemention has been removed from the
    binary, and the option [\--use-fake-symbol-table]{.title-ref} is now
    a no-op with a warning.
-   thrift_proxy: special characters {\'0\', \'r\', \'n\'} will be
    stripped from thrift headers.
-   watchdog: replaced single watchdog with separate watchdog
    configuration for worker threads and for the main thread configured
    via
    `Watchdogs<envoy_v3_api_field_config.bootstrap.v3.Bootstrap.watchdogs>`{.interpreted-text
    role="ref"}. It works with
    `watchdog<envoy_v3_api_field_config.bootstrap.v3.Bootstrap.watchdog>`{.interpreted-text
    role="ref"} by having the worker thread and main thread watchdogs
    have same config.

Bug Fixes
---------

*Changes expected to improve the state of the world and are unlikely to
have negative effects*

-   csrf: fixed issues with regards to origin and host header parsing.
-   dynamic_forward_proxy: only perform DNS lookups for routes to
    Dynamic Forward Proxy clusters since other cluster types handle DNS
    lookup themselves.
-   fault: fixed an issue with [active_faults]{.title-ref} gauge not
    being decremented for when abort faults were injected.
-   fault: made the HeaderNameValues::prefix() method const.
-   grpc-web: fixed an issue with failing HTTP/2 requests on some
    browsers. Notably, WebKit-based browsers
    (<https://bugs.webkit.org/show_bug.cgi?id=210108>), Internet
    Explorer 11, and Edge (pre-Chromium).
-   http: fixed CVE-2020-25018 by rolling back the `GURL` dependency to
    previous state (reverted: `2d69e30`, `d828958`, and `c9c4709`
    commits) due to potential of crashing when Unicode URIs are present
    in requests.
-   http: fixed bugs in datadog and squash filter\'s handling of
    responses with no bodies.
-   http: made the HeaderValues::prefix() method const.
-   jwt_authn: supports jwt payload without \"iss\" field.
-   listener: fixed crash at listener inplace update when connetion load
    balancer is set.
-   rocketmq_proxy: fixed an issue involving incorrect header lengths.
    In debug mode it causes crash and in release mode it causes
    underflow.
-   thrift_proxy: fixed crashing bug on request overflow.
-   udp_proxy: fixed a crash due to UDP packets being processed after
    listener removal.

Removed Config or Runtime
-------------------------

*Normally occurs at the end of the*
`deprecation period <deprecated>`{.interpreted-text role="ref"}

-   http: removed legacy header sanitization and the runtime guard
    [envoy.reloadable_features.strict_header_validation]{.title-ref}.
-   http: removed legacy transfer-encoding enforcement and runtime guard
    [envoy.reloadable_features.reject_unsupported_transfer_encodings]{.title-ref}.
-   http: removed configurable strict host validation and runtime guard
    [envoy.reloadable_features.strict_authority_validation]{.title-ref}.
-   http: removed the connection header sanitization runtime guard
    [envoy.reloadable_features.connection_header_sanitization]{.title-ref}.

New Features
------------

-   access log: added a
    `dynamic metadata filter<envoy_v3_api_msg_config.accesslog.v3.MetadataFilter>`{.interpreted-text
    role="ref"} for access logs, which filters whether to log based on
    matching dynamic metadata.
-   access log: added support for
    `%DOWNSTREAM_PEER_FINGERPRINT_1% <config_access_log_format_response_flags>`{.interpreted-text
    role="ref"} as a response flag.
-   access log: added support for
    `%CONNECTION_TERMINATION_DETAILS% <config_access_log_format_connection_termination_details>`{.interpreted-text
    role="ref"} as a log command operator about why the connection is
    terminated by Envoy.
-   access log: added support for nested objects in
    `JSON logging mode <config_access_log_format_dictionaries>`{.interpreted-text
    role="ref"}.
-   access log: added
    `omit_empty_values<envoy_v3_api_field_config.core.v3.SubstitutionFormatString.omit_empty_values>`{.interpreted-text
    role="ref"} option to omit unset value from formatted log.
-   access log: added support for
    `%CONNECTION_ID% <config_access_log_format_connection_id>`{.interpreted-text
    role="ref"} for the downstream connection identifier.
-   admin: added
    `circuit breakers settings <envoy_v3_api_msg_config.cluster.v3.CircuitBreakers>`{.interpreted-text
    role="ref"} information to GET /clusters?format=json
    `cluster status <envoy_v3_api_msg_admin.v3.ClusterStatus>`{.interpreted-text
    role="ref"}.
-   admin: added
    `node <envoy_v3_api_msg_config.core.v3.Node>`{.interpreted-text
    role="ref"} information to GET /server_info
    `response object <envoy_v3_api_msg_admin.v3.ServerInfo>`{.interpreted-text
    role="ref"}.
-   admin: added the ability to dump init manager unready targets
    information
    `/init_dump <operations_admin_interface_init_dump>`{.interpreted-text
    role="ref"} and
    `/init_dump?mask={} <operations_admin_interface_init_dump_by_mask>`{.interpreted-text
    role="ref"}.
-   admission control: added the
    `admission control <envoy_v3_api_msg_extensions.filters.http.admission_control.v3alpha.AdmissionControl>`{.interpreted-text
    role="ref"} filter for client-side request throttling.
-   build: enable building envoy
    `arm64 images <arm_binaries>`{.interpreted-text role="ref"} by
    buildx tool in x86 CI platform.
-   cluster: added new
    `connection_pool_per_downstream_connection <envoy_v3_api_field_config.cluster.v3.Cluster.connection_pool_per_downstream_connection>`{.interpreted-text
    role="ref"} flag, which enable creation of a new connection pool for
    each downstream connection.
-   decompressor filter: reports compressed and uncompressed bytes in
    trailers.
-   dns: added support for doing DNS resolution using Apple\'s
    DnsService APIs in Apple platforms (macOS, iOS). This feature is ON
    by default, and is only configurable via the
    [envoy.restart_features.use_apple_api_for_dns_lookups]{.title-ref}
    runtime key. Note that this value is latched during server startup
    and changing the runtime key is a no-op during the lifetime of the
    process.
-   dns_filter: added support for answering
    `service record<envoy_v3_api_msg_data.dns.v3.DnsTable.DnsService>`{.interpreted-text
    role="ref"} queries.
-   dynamic_forward_proxy: added
    `use_tcp_for_dns_lookups<envoy_v3_api_field_extensions.common.dynamic_forward_proxy.v3.DnsCacheConfig.use_tcp_for_dns_lookups>`{.interpreted-text
    role="ref"} option to use TCP for DNS lookups in order to match the
    DNS options for
    `Clusters<envoy_v3_api_msg_config.cluster.v3.Cluster>`{.interpreted-text
    role="ref"}.
-   ext_authz filter: added support for emitting dynamic metadata for
    both
    `HTTP <config_http_filters_ext_authz_dynamic_metadata>`{.interpreted-text
    role="ref"} and
    `network <config_network_filters_ext_authz_dynamic_metadata>`{.interpreted-text
    role="ref"} filters. The emitted dynamic metadata is set by
    `dynamic metadata <envoy_v3_api_field_service.auth.v3.CheckResponse.dynamic_metadata>`{.interpreted-text
    role="ref"} field in a returned
    `CheckResponse <envoy_v3_api_msg_service.auth.v3.CheckResponse>`{.interpreted-text
    role="ref"}.
-   ext_authz filter: added
    `stat_prefix <envoy_v3_api_field_extensions.filters.http.ext_authz.v3.ExtAuthz.stat_prefix>`{.interpreted-text
    role="ref"} as an optional additional prefix for the statistics
    emitted from [ext_authz]{.title-ref} HTTP filter.
-   ext_authz filter: added support for enabling the filter based on
    `dynamic metadata <envoy_v3_api_field_extensions.filters.http.ext_authz.v3.ExtAuthz.filter_enabled_metadata>`{.interpreted-text
    role="ref"}.
-   ext_authz filter: added support for letting the authorization server
    instruct Envoy to remove headers from the original request by
    setting the new field
    `headers_to_remove <envoy_v3_api_field_service.auth.v3.OkHttpResponse.headers_to_remove>`{.interpreted-text
    role="ref"} before forwarding it to the upstream.
-   ext_authz filter: added support for sending
    `raw bytes as request body <envoy_v3_api_field_service.auth.v3.AttributeContext.HttpRequest.raw_body>`{.interpreted-text
    role="ref"} of a gRPC check request by setting
    `pack_as_bytes <envoy_v3_api_field_extensions.filters.http.ext_authz.v3.BufferSettings.pack_as_bytes>`{.interpreted-text
    role="ref"} to true.
-   ext_authz_filter: added
    `disable_request_body_buffering <envoy_v3_api_field_extensions.filters.http.ext_authz.v3.CheckSettings.disable_request_body_buffering>`{.interpreted-text
    role="ref"} to disable request data buffering per-route.
-   grpc-json: support specifying [response_body]{.title-ref} field in
    for [google.api.HttpBody]{.title-ref} message.
-   hds: added
    `cluster_endpoints_health <envoy_v3_api_field_service.health.v3.EndpointHealthResponse.cluster_endpoints_health>`{.interpreted-text
    role="ref"} to HDS responses, keeping endpoints in the same
    groupings as they were configured in the HDS specifier by cluster
    and locality instead of as a flat list.
-   hds: added
    `transport_socket_matches <envoy_v3_api_field_service.health.v3.ClusterHealthCheck.transport_socket_matches>`{.interpreted-text
    role="ref"} to HDS cluster health check specifier, so the existing
    match filter
    `transport_socket_match_criteria <envoy_v3_api_field_config.core.v3.HealthCheck.transport_socket_match_criteria>`{.interpreted-text
    role="ref"} in the repeated field
    `health_checks <envoy_v3_api_field_service.health.v3.ClusterHealthCheck.health_checks>`{.interpreted-text
    role="ref"} has context to match against. This unblocks support for
    health checks over HTTPS and HTTP/2.
-   hot restart: added `--socket-path`{.interpreted-text role="option"}
    and `--socket-mode`{.interpreted-text role="option"} to configure
    UDS path in the filesystem and set permission to it.
-   http: added HTTP/2 support for
    `connection keepalive <envoy_v3_api_field_config.core.v3.Http2ProtocolOptions.connection_keepalive>`{.interpreted-text
    role="ref"} via PING.
-   http: added support for
    `%DOWNSTREAM_PEER_FINGERPRINT_1% <config_http_conn_man_headers_custom_request_headers>`{.interpreted-text
    role="ref"} as custom header.
-   http: added
    `allow_chunked_length <envoy_v3_api_field_config.core.v3.Http1ProtocolOptions.allow_chunked_length>`{.interpreted-text
    role="ref"} configuration option for HTTP/1 codec to allow
    processing requests/responses with both Content-Length and
    Transfer-Encoding: chunked headers. If such message is served and
    option is enabled - per RFC Content-Length is ignored and removed.
-   http: added
    `CDN Loop filter <envoy_v3_api_msg_extensions.filters.http.cdn_loop.v3alpha.CdnLoopConfig>`{.interpreted-text
    role="ref"} and
    `documentation <config_http_filters_cdn_loop>`{.interpreted-text
    role="ref"}.
-   http: added
    `MaxStreamDuration proto <envoy_v3_api_msg_config.route.v3.RouteAction.MaxStreamDuration>`{.interpreted-text
    role="ref"} for configuring per-route downstream duration timeouts.
-   http: introduced new HTTP/1 and HTTP/2 codec implementations that
    will remove the use of exceptions for control flow due to high risk
    factors and instead use error statuses. The old behavior is used by
    default for HTTP/1.1 and HTTP/2 server connections. The new codecs
    can be enabled for testing by setting the runtime feature
    [envoy.reloadable_features.new_codec_behavior]{.title-ref} to true.
    The new codecs will be in development for one month, and then
    enabled by default while the old codecs are deprecated.
-   http: modified the HTTP header-map data-structure to use an
    underlying dictionary and a list (no change to the header-map API).
    To conform with previous versions, the use of a dictionary is
    currently disabled. It can be enabled by setting the
    [envoy.http.headermap.lazy_map_min_size]{.title-ref} runtime feature
    to a non-negative number which defines the minimal number of headers
    in a request/response/trailers required for using a dictionary in
    addition to the list. Our current benchmarks suggest that the value
    3 is a good threshold for most workloads.
-   load balancer: added
    `RingHashLbConfig<envoy_v3_api_msg_config.cluster.v3.Cluster.MaglevLbConfig>`{.interpreted-text
    role="ref"} to configure the table size of Maglev consistent hash.
-   load balancer: added a
    `configuration<envoy_v3_api_msg_config.cluster.v3.Cluster.LeastRequestLbConfig>`{.interpreted-text
    role="ref"} option to specify the active request bias used by the
    least request load balancer.
-   load balancer: added an
    `option <envoy_v3_api_field_config.cluster.v3.Cluster.LbSubsetConfig.LbSubsetSelector.single_host_per_subset>`{.interpreted-text
    role="ref"} to optimize subset load balancing when there is only one
    host per subset.
-   load balancer: added support for bounded load per host for
    consistent hash load balancers via
    `hash_balance_factor <envoy_api_field_Cluster.CommonLbConfig.consistent_hashing_lb_config>`{.interpreted-text
    role="ref"}.
-   local_ratelimit: added new
    `HTTP local ratelimit filter <config_http_filters_local_rate_limit>`{.interpreted-text
    role="ref"}.
-   local_reply config: added
    `content_type<envoy_v3_api_field_config.core.v3.SubstitutionFormatString.content_type>`{.interpreted-text
    role="ref"} field to set content-type.
-   lua: added Lua APIs to access
    `SSL connection info <config_http_filters_lua_ssl_socket_info>`{.interpreted-text
    role="ref"} object.
-   lua: added Lua API for
    `base64 escaping a string <config_http_filters_lua_stream_handle_api_base64_escape>`{.interpreted-text
    role="ref"}.
-   lua: added Lua API for
    `setting the current buffer content <config_http_filters_lua_buffer_wrapper_api_set_bytes>`{.interpreted-text
    role="ref"}.
-   lua: added new
    `source_code <envoy_v3_api_field_extensions.filters.http.lua.v3.LuaPerRoute.source_code>`{.interpreted-text
    role="ref"} field to support the dispatching of inline Lua code in
    per route configuration of Lua filter.
-   overload management: add
    `scaling <envoy_v3_api_field_config.overload.v3.Trigger.scaled>`{.interpreted-text
    role="ref"} trigger for OverloadManager actions.
-   postgres network filter:
    `metadata <config_network_filters_postgres_proxy_dynamic_metadata>`{.interpreted-text
    role="ref"} is produced based on SQL query.
-   proxy protocol: added support for generating the header upstream
    using
    `Proxy Protocol Transport Socket <extension_envoy.transport_sockets.upstream_proxy_protocol>`{.interpreted-text
    role="ref"}.
-   ratelimit: added
    `enable_x_ratelimit_headers <envoy_v3_api_msg_extensions.filters.http.ratelimit.v3.RateLimit>`{.interpreted-text
    role="ref"} option to enable [X-RateLimit-\*]{.title-ref} headers as
    defined in [draft
    RFC](https://tools.ietf.org/id/draft-polli-ratelimit-headers-03.html).
-   ratelimit: added
    `per route config <envoy_v3_api_msg_extensions.filters.http.ratelimit.v3.RateLimitPerRoute>`{.interpreted-text
    role="ref"} for rate limit filter.
-   ratelimit: added support for optional
    `descriptor_key <envoy_v3_api_field_config.route.v3.RateLimit.Action.generic_key>`{.interpreted-text
    role="ref"} to Generic Key action.
-   rbac filter: added the name of the matched policy to the response
    code detail when a request is rejected by the RBAC filter.
-   rbac filter: added a log action to the
    `RBAC filter <envoy_v3_api_msg_config.rbac.v3.RBAC>`{.interpreted-text
    role="ref"} which sets dynamic metadata to inform access loggers
    whether to log.
-   redis: added fault injection support
    `fault injection for redis proxy <envoy_v3_api_field_extensions.filters.network.redis_proxy.v3.RedisProxy.faults>`{.interpreted-text
    role="ref"}, described further in
    `configuration documentation <config_network_filters_redis_proxy>`{.interpreted-text
    role="ref"}.
-   router: added a new
    `rate limited retry back off <envoy_v3_api_msg_config.route.v3.RetryPolicy.RateLimitedRetryBackOff>`{.interpreted-text
    role="ref"} strategy that uses headers like
    [Retry-After]{.title-ref} or [X-RateLimit-Reset]{.title-ref} to
    decide the back off interval.
-   router: added new
    `envoy-ratelimited<config_http_filters_router_retry_policy-envoy-ratelimited>`{.interpreted-text
    role="ref"} retry policy, which allows retrying envoy\'s own rate
    limited responses.
-   router: added new
    `host_rewrite_path_regex <envoy_v3_api_field_config.route.v3.RouteAction.host_rewrite_path_regex>`{.interpreted-text
    role="ref"} option, which allows rewriting Host header based on
    path.
-   router: added support for DYNAMIC_METADATA
    `header formatter <config_http_conn_man_headers_custom_request_headers>`{.interpreted-text
    role="ref"}.
-   router_check_tool: added support for
    [request_header_matches]{.title-ref},
    [response_header_matches]{.title-ref} to
    `router check tool <config_tools_router_check_tool>`{.interpreted-text
    role="ref"}.
-   signal: added support for calling fatal error handlers without
    envoy\'s signal handler, via
    FatalErrorHandler::callFatalErrorHandlers().
-   stats: added optional histograms to
    `cluster stats <config_cluster_manager_cluster_stats_request_response_sizes>`{.interpreted-text
    role="ref"} that track headers and body sizes of requests and
    responses.
-   stats: allow configuring histogram buckets for stats sinks and admin
    endpoints that support it.
-   tap: added
    `generic body matcher<envoy_v3_api_msg_config.tap.v3.HttpGenericBodyMatch>`{.interpreted-text
    role="ref"} to scan http requests and responses for text or hex
    patterns.
-   tcp_proxy: added
    `max_downstream_connection_duration<envoy_v3_api_field_extensions.filters.network.tcp_proxy.v3.TcpProxy.max_downstream_connection_duration>`{.interpreted-text
    role="ref"} for downstream connection. When max duration is reached
    the connection will be closed.
-   tcp_proxy: allow earlier network filters to set
    metadataMatchCriteria on the connection StreamInfo to influence load
    balancing.
-   tls: added OCSP stapling support through the
    `ocsp_staple <envoy_v3_api_msg_extensions.transport_sockets.tls.v3.TlsCertificate>`{.interpreted-text
    role="ref"} and
    `ocsp_staple_policy <envoy_v3_api_msg_extensions.transport_sockets.tls.v3.DownstreamTlsContext>`{.interpreted-text
    role="ref"} configuration options. See
    `OCSP Stapling <arch_overview_ssl_ocsp_stapling>`{.interpreted-text
    role="ref"} for usage and runtime flags.
-   tls: introduce new
    `extension point<envoy_v3_api_field_extensions.transport_sockets.tls.v3.CommonTlsContext.custom_handshaker>`{.interpreted-text
    role="ref"} for overriding
    `TLS handshaker <arch_overview_ssl>`{.interpreted-text role="ref"}
    behavior.
-   tls: switched from using socket BIOs to using custom BIOs that know
    how to interact with IoHandles. The feature can be disabled by
    setting runtime feature
    [envoy.reloadable_features.tls_use_io_handle_bio]{.title-ref} to
    false.
-   tracing: added ability to set some
    `optional segment fields<envoy_v3_api_field_config.trace.v3.XRayConfig.segment_fields>`{.interpreted-text
    role="ref"} in the AWS X-Ray tracer.
-   udp_proxy: added
    `hash_policies <envoy_v3_api_msg_extensions.filters.udp.udp_proxy.v3.UdpProxyConfig>`{.interpreted-text
    role="ref"} to support hash based routing.
-   udp_proxy: added
    `use_original_src_ip <envoy_v3_api_msg_extensions.filters.udp.udp_proxy.v3.UdpProxyConfig>`{.interpreted-text
    role="ref"} option to replicate the downstream remote address of the
    packets on the upstream side of Envoy. It is similar to
    `original source filter <envoy_v3_api_msg_extensions.filters.listener.original_src.v3.OriginalSrc>`{.interpreted-text
    role="ref"}.
-   watchdog: support randomizing the watchdog\'s kill timeout to
    prevent synchronized kills via a maximium jitter parameter
    `max_kill_timeout_jitter<envoy_v3_api_field_config.bootstrap.v3.Watchdog.max_kill_timeout_jitter>`{.interpreted-text
    role="ref"}.
-   watchdog: supports an extension point where actions can be
    registered to fire on watchdog events such as miss, megamiss, kill
    and multikill. See
    `watchdog actions<envoy_v3_api_field_config.bootstrap.v3.Watchdog.actions>`{.interpreted-text
    role="ref"}.
-   watchdog: watchdog action extension that does cpu profiling. See
    `Profile Action <envoy_v3_api_file_envoy/extensions/watchdog/profile_action/v3alpha/profile_action.proto>`{.interpreted-text
    role="ref"}.
-   watchdog: watchdog action extension that sends SIGABRT to the stuck
    thread to terminate the process. See
    `Abort Action <envoy_v3_api_file_envoy/extensions/watchdog/abort_action/v3alpha/abort_action.proto>`{.interpreted-text
    role="ref"}.
-   xds: added
    `extension config discovery<envoy_v3_api_msg_config.core.v3.ExtensionConfigSource>`{.interpreted-text
    role="ref"} support for HTTP filters.
-   xds: added support for mixed v2/v3 discovery response, which enable
    type url downgrade and upgrade. This feature is disabled by default
    and is controlled by runtime guard
    [envoy.reloadable_features.enable_type_url_downgrade_and_upgrade]{.title-ref}.
-   zlib: added option to use
    [zlib-ng](https://github.com/zlib-ng/zlib-ng) as zlib library.

Deprecated
----------

-   build: alpine based debug image is deprecated in favor of
    `Ubuntu based debug image <install_binaries>`{.interpreted-text
    role="ref"}.
-   cluster: the
    `track_timeout_budgets <envoy_v3_api_field_config.cluster.v3.Cluster.track_timeout_budgets>`{.interpreted-text
    role="ref"} field has been deprecated in favor of
    [timeout_budgets]{.title-ref} part of an
    `Optional Configuration <envoy_v3_api_field_config.cluster.v3.Cluster.track_cluster_stats>`{.interpreted-text
    role="ref"}.
-   ext_authz: the
    `dynamic metadata <envoy_v3_api_field_service.auth.v3.OkHttpResponse.dynamic_metadata>`{.interpreted-text
    role="ref"} field in
    `OkHttpResponse <envoy_v3_api_msg_service.auth.v3.OkHttpResponse>`{.interpreted-text
    role="ref"} has been deprecated in favor of
    `dynamic metadata <envoy_v3_api_field_service.auth.v3.CheckResponse.dynamic_metadata>`{.interpreted-text
    role="ref"} field in
    `CheckResponse <envoy_v3_api_msg_service.auth.v3.CheckResponse>`{.interpreted-text
    role="ref"}.
-   hds: the
    `endpoints_health <envoy_v3_api_field_service.health.v3.EndpointHealthResponse.endpoints_health>`{.interpreted-text
    role="ref"} field has been deprecated in favor of
    `cluster_endpoints_health <envoy_v3_api_field_service.health.v3.EndpointHealthResponse.cluster_endpoints_health>`{.interpreted-text
    role="ref"} to maintain grouping by cluster and locality.
-   router: the
    `include_vh_rate_limits <envoy_v3_api_field_config.route.v3.RouteAction.include_vh_rate_limits>`{.interpreted-text
    role="ref"} field has been deprecated in favor of
    `vh_rate_limits <envoy_v3_api_field_extensions.filters.http.ratelimit.v3.RateLimitPerRoute.vh_rate_limits>`{.interpreted-text
    role="ref"}.
-   router: the
    `max_grpc_timeout <envoy_v3_api_field_config.route.v3.RouteAction.max_grpc_timeout>`{.interpreted-text
    role="ref"} field has been deprecated in favor of
    `grpc_timeout_header_max <envoy_v3_api_field_config.route.v3.RouteAction.MaxStreamDuration.grpc_timeout_header_max>`{.interpreted-text
    role="ref"}.
-   router: the
    `grpc_timeout_offset <envoy_v3_api_field_config.route.v3.RouteAction.grpc_timeout_offset>`{.interpreted-text
    role="ref"} field has been deprecated in favor of
    `grpc_timeout_header_offset <envoy_v3_api_field_config.route.v3.RouteAction.MaxStreamDuration.grpc_timeout_header_offset>`{.interpreted-text
    role="ref"}.
-   tap: the
    `match_config <envoy_v3_api_field_config.tap.v3.TapConfig.match_config>`{.interpreted-text
    role="ref"} field has been deprecated in favor of
    `match <envoy_v3_api_field_config.tap.v3.TapConfig.match>`{.interpreted-text
    role="ref"} field.
-   router_check_tool: [request_header_fields]{.title-ref},
    [response_header_fields]{.title-ref} config deprecated in favor of
    [request_header_matches]{.title-ref},
    [response_header_matches]{.title-ref}.
-   watchdog:
    `watchdog <envoy_v3_api_field_config.bootstrap.v3.Bootstrap.watchdog>`{.interpreted-text
    role="ref"} deprecated in favor of
    `watchdogs <envoy_v3_api_field_config.bootstrap.v3.Bootstrap.watchdogs>`{.interpreted-text
    role="ref"}.
