AWS Lambda {#config_http_filters_aws_lambda}
==========

-   `v3 API reference <envoy_v3_api_msg_extensions.filters.http.aws_lambda.v3.Config>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.aws_lambda*.

::: {.attention}
::: {.title}
Attention
:::

The AWS Lambda filter is currently under active development.
:::

The HTTP AWS Lambda filter is used to trigger an AWS Lambda function
from a standard HTTP/1.x or HTTP/2 request. It supports a few options to
control whether to pass through the HTTP request payload as is or to
wrap it in a JSON schema.

If
`payload_passthrough <envoy_v3_api_field_extensions.filters.http.aws_lambda.v3.Config.payload_passthrough>`{.interpreted-text
role="ref"} is set to `true`, then the payload is sent to Lambda without
any transformations. *Note*: This means you lose access to all the HTTP
headers in the Lambda function.

However, if
`payload_passthrough <envoy_v3_api_field_extensions.filters.http.aws_lambda.v3.Config.payload_passthrough>`{.interpreted-text
role="ref"} is set to `false`, then the HTTP request is transformed to a
JSON payload with the following schema:

``` {.}
{
    "rawPath": "/path/to/resource",
    "method": "GET|POST|HEAD|...",
    "headers": {"header-key": "header-value", ... },
    "queryStringParameters": {"key": "value", ...},
    "body": "...",
    "isBase64Encoded": true|false
}
```

-   `rawPath` is the HTTP request resource path (including the query
    string)

-   `method` is the HTTP request method. For example `GET`, `PUT`, etc.

-   `headers` are the HTTP request headers. If multiple headers share
    the same name, their values are coalesced into a single
    comma-separated value.

-   `queryStringParameters` are the HTTP request query string
    parameters. If multiple parameters share the same name, the last one
    wins. That is, parameters are \_[not]() coalesced into a single
    value if they share the same key name.

-   `body` the body of the HTTP request is base64-encoded by the filter
    if the `content-type` header exists and is \_[not]() one of the
    following:

    > -   text/\*
    > -   application/json
    > -   application/xml
    > -   application/javascript

Otherwise, the body of HTTP request is added to the JSON payload as is.

On the other end, the response of the Lambda function must conform to
the following schema:

``` {.}
{
    "statusCode": ...
    "headers": {"header-key": "header-value", ... },
    "cookies": ["key1=value1; HttpOnly; ...", "key2=value2; Secure; ...", ...],
    "body": "...",
    "isBase64Encoded": true|false
}
```

-   The `statusCode` field is an integer used as the HTTP response code.
    If this key is missing, Envoy returns a `200 OK`.
-   The `headers` are used as the HTTP response headers.
-   The `cookies` are used as `Set-Cookie` response headers. Unlike the
    request headers, cookies are \_[not]() part of the response headers
    because the `Set-Cookie` header cannot contain more than one value
    per the [RFC](https://tools.ietf.org/html/rfc6265#section-4.1).
    Therefore, Each key/value pair in this JSON array will translate to
    a single `Set-Cookie` header.
-   The `body` is base64-decoded if it is marked as base64-encoded and
    sent as the body of the HTTP response.

::: {.note}
::: {.title}
Note
:::

The target cluster must have its endpoint set to the [regional Lambda
endpoint](https://docs.aws.amazon.com/general/latest/gr/lambda-service.html).
Use the same region as the Lambda function.

AWS IAM credentials must be defined in either environment variables, EC2
metadata or ECS task metadata.
:::

The filter supports `per-filter configuration
<envoy_v3_api_msg_extensions.filters.http.aws_lambda.v3.PerRouteConfig>`{.interpreted-text
role="ref"}.

If you use the per-filter configuration, the target cluster \_[must]()
have the following metadata:

``` {.yaml}
metadata:
  filter_metadata:
    com.amazonaws.lambda:
      egress_gateway: true
```

Below are some examples that show how the filter can be used in
different deployment scenarios.

Example configuration
---------------------

In this configuration, the filter applies to all routes in the filter
chain of the http connection manager:

``` {.yaml}
http_filters:
- name: envoy.filters.http.aws_lambda
  typed_config:
    "@type": type.googleapis.com/envoy.extensions.filters.http.aws_lambda.v3.Config
    arn: "arn:aws:lambda:us-west-2:987654321:function:hello_envoy"
    payload_passthrough: true
```

The corresponding regional endpoint must be specified in the target
cluster. So, for example if the Lambda function is in us-west-2:

``` {.yaml}
clusters:
- name: lambda_egress_gateway
  connect_timeout: 0.25s
  type: LOGICAL_DNS
  dns_lookup_family: V4_ONLY
  lb_policy: ROUND_ROBIN
  load_assignment:
    cluster_name: lambda_egress_gateway
    endpoints:
    - lb_endpoints:
      - endpoint:
          address:
            socket_address:
              address: lambda.us-west-2.amazonaws.com
              port_value: 443
  transport_socket:
    name: envoy.transport_sockets.tls
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.UpstreamTlsContext
      sni: "*.amazonaws.com"
```

The filter can also be configured per virtual-host, route or
weighted-cluster. In that case, the target cluster *must* have specific
Lambda metadata.

``` {.yaml}
weighted_clusters:
clusters:
- name: lambda_egress_gateway
  weight: 42
  typed_per_filter_config:
    envoy.filters.http.aws_lambda:
      "@type": type.googleapis.com/envoy.extensions.filters.http.aws_lambda.v3.PerRouteConfig
      invoke_config:
        arn: "arn:aws:lambda:us-west-2:987654321:function:hello_envoy"
        payload_passthrough: false
```

An example with the Lambda metadata applied to a weighted-cluster:

``` {.yaml}
clusters:
- name: lambda_egress_gateway
  connect_timeout: 0.25s
  type: LOGICAL_DNS
  dns_lookup_family: V4_ONLY
  lb_policy: ROUND_ROBIN
  metadata:
    filter_metadata:
      com.amazonaws.lambda:
        egress_gateway: true
  load_assignment:
    cluster_name: lambda_egress_gateway # does this have to match? seems redundant
    endpoints:
    - lb_endpoints:
      - endpoint:
          address:
            socket_address:
              address: lambda.us-west-2.amazonaws.com
              port_value: 443
  transport_socket:
    name: envoy.transport_sockets.tls
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.UpstreamTlsContext
      sni: "*.amazonaws.com"
```

Statistics
----------

The AWS Lambda filter outputs statistics in the
*http.\<stat_prefix\>.aws_lambda.* namespace. The
`stat prefix <envoy_api_field_config.filter.network.http_connection_manager.v2.HttpConnectionManager.stat_prefix>`{.interpreted-text
role="ref"} comes from the owning HTTP connection manager.

  ---------------------------------------------------------------------------------------------------------------------------------------------------
  Name                       Type              Description
  -------------------------- ----------------- ------------------------------------------------------------------------------------------------------
  server_error               Counter           Total requests that returned invalid JSON response (see
                                               `payload_passthrough <envoy_api_msg_config.filter.http.aws_lambda.v2alpha.config>`{.interpreted-text
                                               role="ref"})

  upstream_rq_payload_size   Histogram         Size in bytes of the request after JSON-tranformation (if any).
  ---------------------------------------------------------------------------------------------------------------------------------------------------
