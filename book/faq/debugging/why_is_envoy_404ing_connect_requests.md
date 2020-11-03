Why is Envoy sending 404s to CONNECT requests? {#faq_why_is_envoy_404ing_connect_requests}
==============================================

Envoy\'s default matchers match based on host and path. Because CONNECT
requests (generally) do not have a path, most matchers will fail to
match CONNECT requests, and Envoy will send a 404 because the route is
not found. The solution for HTTP/1.1 CONNECT requests, is to use a
`connect_matcher <envoy_v3_api_msg_config.route.v3.RouteMatch.ConnectMatcher>`{.interpreted-text
role="ref"} as described in the CONNECT section of the
`upgrade documentation<arch_overview_upgrades>`{.interpreted-text
role="ref"}.
