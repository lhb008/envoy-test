How are configuration deprecations handled? {#faq_deprecation}
===========================================

As documented in the \"Breaking change policy\" in
`CONTRIBUTING.md`{.interpreted-text role="repo"}, features can be marked
as deprecated at any point as long as there is a replacement available.
Each deprecation is annotated in the API proto itself and explained in
detail in the `Envoy documentation <deprecated>`{.interpreted-text
role="ref"}.

For the first 3 months following deprecation, use of deprecated fields
will result in a logged warning and incrementing the
`deprecated_feature_use <runtime_stats>`{.interpreted-text role="ref"}
counter. After that point, the field will be annotated as
fatal-by-default and further use of the field will be treated as invalid
configuration unless
`runtime overrides <config_runtime_deprecation>`{.interpreted-text
role="ref"} are employed to re-enable use.
