gRPC-JSON transcoder {#config_http_filters_grpc_json_transcoder}
====================

-   gRPC `architecture overview <arch_overview_grpc>`{.interpreted-text
    role="ref"}
-   `v3 API reference <envoy_v3_api_msg_extensions.filters.http.grpc_json_transcoder.v3.GrpcJsonTranscoder>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.grpc_json_transcoder*.

This is a filter which allows a RESTful JSON API client to send requests
to Envoy over HTTP and get proxied to a gRPC service. The HTTP mapping
for the gRPC service has to be defined by [custom
options](https://cloud.google.com/service-management/reference/rpc/google.api#http).

JSON mapping
------------

The protobuf to JSON mapping is defined
[here](https://developers.google.com/protocol-buffers/docs/proto3#json).
For gRPC stream request parameters, Envoy expects an array of messages,
and it returns an array of messages for stream response parameters.

How to generate proto descriptor set {#config_grpc_json_generate_proto_descriptor_set}
------------------------------------

Envoy has to know the proto descriptor of your gRPC service in order to
do the transcoding.

To generate a protobuf descriptor set for the gRPC service, you\'ll also
need to clone the googleapis repository from GitHub before running
protoc, as you\'ll need annotations.proto in your include path, to
define the HTTP mapping.

``` {.bash}
git clone https://github.com/googleapis/googleapis
GOOGLEAPIS_DIR=<your-local-googleapis-folder>
```

Then run protoc to generate the descriptor set from bookstore.proto:

``` {.bash}
protoc -I$(GOOGLEAPIS_DIR) -I. --include_imports --include_source_info \
  --descriptor_set_out=proto.pb test/proto/bookstore.proto
```

If you have more than one proto source files, you can pass all of them
in one command.

Route configs for transcoded requests
-------------------------------------

The route configs to be used with the gRPC-JSON transcoder should be
identical to the gRPC route. The requests processed by the transcoder
filter will have [/\<package\>.\<service\>/\<method\>]{.title-ref} path
and [POST]{.title-ref} method. The route configs for those requests
should match on [/\<package\>.\<service\>/\<method\>]{.title-ref}, not
the incoming request path. This allows the routes to be used for both
gRPC requests and gRPC-JSON transcoded requests.

For example, with the following proto example, the router will process
[/helloworld.Greeter/SayHello]{.title-ref} as the path, so the route
config prefix [/say]{.title-ref} won\'t match requests to
[SayHello]{.title-ref}. If you want to match the incoming request path,
set [match_incoming_request_route]{.title-ref} to true.

``` {.proto}
package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {
    option (google.api.http) = {
      get: "/say"
    };
  }
}
```

Sending arbitrary content
-------------------------

By default, when transcoding occurs, gRPC-JSON encodes the message
output of a gRPC service method into JSON and sets the HTTP response
[Content-Type]{.title-ref} header to [application/json]{.title-ref}. To
send arbitrary content, a gRPC service method can use
[google.api.HttpBody](https://github.com/googleapis/googleapis/blob/master/google/api/httpbody.proto)
as its output message type. The implementation needs to set
[content_type](https://github.com/googleapis/googleapis/blob/master/google/api/httpbody.proto#L68)
(which sets the value of the HTTP response [Content-Type]{.title-ref}
header) and
[data](https://github.com/googleapis/googleapis/blob/master/google/api/httpbody.proto#L71)
(which sets the HTTP response body) accordingly. Multiple
[google.api.HttpBody](https://github.com/googleapis/googleapis/blob/master/google/api/httpbody.proto)
can be send by the gRPC server in the server streaming case. In this
case, HTTP response header [Content-Type]{.title-ref} will use the
[content-type]{.title-ref} from the first
[google.api.HttpBody](https://github.com/googleapis/googleapis/blob/master/google/api/httpbody.proto).

Headers
-------

gRPC-JSON forwards the following headers to the gRPC server:

-   [x-envoy-original-path]{.title-ref}, containing the value of the
    original path of HTTP request
-   [x-envoy-original-method]{.title-ref}, containing the value of the
    original method of HTTP request

Sample Envoy configuration
--------------------------

Here\'s a sample Envoy configuration that proxies to a gRPC server
running on localhost:50051. Port 51051 proxies gRPC requests and uses
the gRPC-JSON transcoder filter to provide the RESTful JSON mapping.
I.e., you can make either gRPC or RESTful JSON requests to
localhost:51051.

::: {.literalinclude language="yaml"}
\_include/grpc-transcoder-filter.yaml
:::
