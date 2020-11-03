Schema Validator check tool {#install_tools_schema_validator_check_tool}
===========================

The schema validator tool validates that the passed in configuration
conforms to a given schema. The configuration may be JSON or YAML. To
validate the entire config, please refer to the
`config load check tool<install_tools_config_load_check_tool>`{.interpreted-text
role="ref"}.

Input

:   The tool expects two inputs:

    1.  The schema type to check the passed in configuration against.
        The supported types are:

    > -   [route]{.title-ref} - for
    >     `route configuration<envoy_v3_api_msg_config.route.v3.RouteConfiguration>`{.interpreted-text
    >     role="ref"} validation.
    > -   [discovery_response]{.title-ref} for
    >     `discovery response<envoy_v3_api_msg_service.discovery.v3.DiscoveryResponse>`{.interpreted-text
    >     role="ref"} validation.

    2.  The path to the configuration file.

Output

:   If the configuration conforms to the schema, the tool will exit with
    status EXIT_SUCCESS. If the configuration does not conform to the
    schema, an error message is outputted detailing what doesn\'t
    conform to the schema. The tool will exit with status EXIT_FAILURE.

Building

:   The tool can be built locally using Bazel. :

        bazel build //test/tools/schema_validator:schema_validator_tool

Running

:   The tool takes a path as described above. :

        bazel-bin/test/tools/schema_validator/schema_validator_tool  --schema-type SCHEMA_TYPE  --config-path PATH
