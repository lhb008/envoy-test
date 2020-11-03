Lua {#config_http_filters_lua}
===

::: {.attention}
::: {.title}
Attention
:::

By default Envoy is built without exporting symbols that you may need
when interacting with Lua modules installed as shared objects. Envoy may
need to be built with support for exported symbols. Please see the
`Bazel docs <bazel/README.md>`{.interpreted-text role="repo"} for more
information.
:::

Overview
--------

The HTTP Lua filter allows [Lua](https://www.lua.org/) scripts to be run
during both the request and response flows.
[LuaJIT](https://luajit.org/) is used as the runtime. Because of this,
the supported Lua version is mostly 5.1 with some 5.2 features. See the
[LuaJIT documentation](https://luajit.org/extensions.html) for more
details.

::: {.note}
::: {.title}
Note
:::

[moonjit](https://github.com/moonjit/moonjit/) is a continuation of
LuaJIT development, which supports more 5.2 features and additional
architectures. Envoy can be built with moonjit support by using the
following bazel option:
`--//source/extensions/filters/common/lua:moonjit=1`.
:::

The design of the filter and Lua support at a high level is as follows:

-   All Lua environments are
    `per worker thread <arch_overview_threading>`{.interpreted-text
    role="ref"}. This means that there is no truly global data. Any
    globals created and populated at load time will be visible from each
    worker thread in isolation. True global support may be added via an
    API in the future.
-   All scripts are run as coroutines. This means that they are written
    in a synchronous style even though they may perform complex
    asynchronous tasks. This makes the scripts substantially easier to
    write. All network/async processing is performed by Envoy via a set
    of APIs. Envoy will suspend execution of the script as appropriate
    and resume it when async tasks are complete.
-   **Do not perform blocking operations from scripts.** It is critical
    for performance that Envoy APIs are used for all IO.

Currently supported high level features
---------------------------------------

**NOTE:** It is expected that this list will expand over time as the
filter is used in production. The API surface has been kept small on
purpose. The goal is to make scripts extremely simple and safe to write.
Very complex or high performance use cases are assumed to use the native
C++ filter API.

-   Inspection of headers, body, and trailers while streaming in either
    the request flow, response flow, or both.
-   Modification of headers and trailers.
-   Blocking and buffering the full request/response body for
    inspection.
-   Performing an outbound async HTTP call to an upstream host. Such a
    call can be performed while buffering body data so that when the
    call completes upstream headers can be modified.
-   Performing a direct response and skipping further filter iteration.
    For example, a script could make an upstream HTTP call for
    authentication, and then directly respond with a 403 response code.

Configuration
-------------

-   `v3 API reference <envoy_v3_api_msg_extensions.filters.http.lua.v3.Lua>`{.interpreted-text
    role="ref"}
-   This filter should be configured with the name
    *envoy.filters.http.lua*.

A simple example of configuring Lua HTTP filter that contains only
`inline_code
<envoy_v3_api_field_extensions.filters.http.lua.v3.Lua.inline_code>`{.interpreted-text
role="ref"} is as follow:

``` {.yaml}
name: envoy.filters.http.lua
typed_config:
  "@type": type.googleapis.com/envoy.extensions.filters.http.lua.v3.Lua
  inline_code: |
    -- Called on the request path.
    function envoy_on_request(request_handle)
      -- Do something.
    end
    -- Called on the response path.
    function envoy_on_response(response_handle)
      -- Do something.
    end
```

By default, Lua script defined in `inline_code` will be treated as a
`GLOBAL` script. Envoy will execute it for every HTTP request.

Per-Route Configuration
-----------------------

The Lua HTTP filter also can be disabled or overridden on a per-route
basis by providing a
`LuaPerRoute <envoy_v3_api_msg_extensions.filters.http.lua.v3.LuaPerRoute>`{.interpreted-text
role="ref"} configuration on the virtual host, route, or weighted
cluster.

LuaPerRoute provides two ways of overriding the [GLOBAL]{.title-ref} Lua
script:

-   By providing a name reference to the defined
    `named Lua source codes map
    <envoy_v3_api_field_extensions.filters.http.lua.v3.Lua.source_codes>`{.interpreted-text
    role="ref"}.
-   By providing inline `source code
    <envoy_v3_api_field_extensions.filters.http.lua.v3.LuaPerRoute.source_code>`{.interpreted-text
    role="ref"} (This allows the code to be sent through RDS).

As a concrete example, given the following Lua filter configuration:

``` {.yaml}
name: envoy.filters.http.lua
typed_config:
  "@type": type.googleapis.com/envoy.extensions.filters.http.lua.v3.Lua
  inline_code: |
    function envoy_on_request(request_handle)
      -- do something
    end
  source_codes:
    hello.lua:
      inline_string: |
        function envoy_on_request(request_handle)
          request_handle:logInfo("Hello World.")
        end
    bye.lua:
      inline_string: |
        function envoy_on_response(response_handle)
          response_handle:logInfo("Bye Bye.")
        end
```

The HTTP Lua filter can be disabled on some virtual host, route, or
weighted cluster by the
`LuaPerRoute <envoy_v3_api_msg_extensions.filters.http.lua.v3.LuaPerRoute>`{.interpreted-text
role="ref"} configuration as follow:

``` {.yaml}
per_filter_config:
  envoy.filters.http.lua:
    disabled: true
```

We can also refer to a Lua script in the filter configuration by
specifying a name in LuaPerRoute. The `GLOBAL` Lua script will be
overridden by the referenced script:

``` {.yaml}
per_filter_config:
  envoy.filters.http.lua:
    name: hello.lua
```

::: {.attention}
::: {.title}
Attention
:::

The name `GLOBAL` is reserved for `Lua.inline_code
<envoy_v3_api_field_extensions.filters.http.lua.v3.Lua.inline_code>`{.interpreted-text
role="ref"}. Therefore, do not use `GLOBAL` as name for other Lua
scripts.
:::

Or we can define a new Lua script in the LuaPerRoute configuration
directly to override the [GLOBAL]{.title-ref} Lua script as follows:

``` {.yaml}
per_filter_config:
  envoy.filters.http.lua:
    source_code:
      inline_string: |
        function envoy_on_response(response_handle)
          response_handle:logInfo("Goodbye.")
        end
```

Script examples
---------------

This section provides some concrete examples of Lua scripts as a more
gentle introduction and quick start. Please refer to the
`stream handle API <config_http_filters_lua_stream_handle_api>`{.interpreted-text
role="ref"} for more details on the supported API.

``` {.lua}
-- Called on the request path.
function envoy_on_request(request_handle)
  -- Wait for the entire request body and add a request header with the body size.
  request_handle:headers():add("request_body_size", request_handle:body():length())
end

-- Called on the response path.
function envoy_on_response(response_handle)
  -- Wait for the entire response body and add a response header with the body size.
  response_handle:headers():add("response_body_size", response_handle:body():length())
  -- Remove a response header named 'foo'
  response_handle:headers():remove("foo")
end
```

``` {.lua}
function envoy_on_request(request_handle)
  -- Make an HTTP call to an upstream host with the following headers, body, and timeout.
  local headers, body = request_handle:httpCall(
  "lua_cluster",
  {
    [":method"] = "POST",
    [":path"] = "/",
    [":authority"] = "lua_cluster"
  },
  "hello world",
  5000)

  -- Add information from the HTTP call into the headers that are about to be sent to the next
  -- filter in the filter chain.
  request_handle:headers():add("upstream_foo", headers["foo"])
  request_handle:headers():add("upstream_body_size", #body)
end
```

``` {.lua}
function envoy_on_request(request_handle)
  -- Make an HTTP call.
  local headers, body = request_handle:httpCall(
  "lua_cluster",
  {
    [":method"] = "POST",
    [":path"] = "/",
    [":authority"] = "lua_cluster",
    ["set-cookie"] = { "lang=lua; Path=/", "type=binding; Path=/" }
  },
  "hello world",
  5000)

  -- Response directly and set a header from the HTTP call. No further filter iteration
  -- occurs.
  request_handle:respond(
    {[":status"] = "403",
     ["upstream_foo"] = headers["foo"]},
    "nope")
end
```

``` {.lua}
function envoy_on_request(request_handle)
  -- Log information about the request
  request_handle:logInfo("Authority: "..request_handle:headers():get(":authority"))
  request_handle:logInfo("Method: "..request_handle:headers():get(":method"))
  request_handle:logInfo("Path: "..request_handle:headers():get(":path"))
end

function envoy_on_response(response_handle)
  -- Log response status code
  response_handle:logInfo("Status: "..response_handle:headers():get(":status"))
end
```

A common use-case is to rewrite upstream response body, for example: an
upstream sends non-2xx response with JSON data, but the application
requires HTML page to be sent to browsers.

There are two ways of doing this, the first one is via the
[body()]{.title-ref} API.

``` {.lua}
function envoy_on_response(response_handle)
  local content_length = response_handle:body():setBytes("<html><b>Not Found<b></html>")
  response_handle:headers():replace("content-length", content_length)
  response_handle:headers():replace("content-type", "text/html")
end
```

Or, through [bodyChunks()]{.title-ref} API, which let Envoy to skip
buffering the upstream response data.

``` {.lua}
function envoy_on_response(response_handle)

  -- Sets the content-length.
  response_handle:headers():replace("content-length", 28)
  response_handle:headers():replace("content-type", "text/html")

  local last
  for chunk in response_handle:bodyChunks() do
    -- Clears each received chunk.
    chunk:setBytes("")
    last = chunk
  end

  last:setBytes("<html><b>Not Found<b></html>")
end
```

Complete example {#config_http_filters_lua_stream_handle_api}
----------------

A complete example using Docker is available in
`/examples/lua`{.interpreted-text role="repo"}.

Stream handle API
-----------------

When Envoy loads the script in the configuration, it looks for two
global functions that the script defines:

``` {.lua}
function envoy_on_request(request_handle)
end

function envoy_on_response(response_handle)
end
```

A script can define either or both of these functions. During the
request path, Envoy will run *envoy_on_request* as a coroutine, passing
a handle to the request API. During the response path, Envoy will run
*envoy_on_response* as a coroutine, passing handle to the response API.

::: {.attention}
::: {.title}
Attention
:::

It is critical that all interaction with Envoy occur through the passed
stream handle. The stream handle should not be assigned to any global
variable and should not be used outside of the coroutine. Envoy will
fail your script if the handle is used incorrectly.
:::

The following methods on the stream handle are supported:

### headers()

``` {.lua}
local headers = handle:headers()
```

Returns the stream\'s headers. The headers can be modified as long as
they have not been sent to the next filter in the header chain. For
example, they can be modified after an *httpCall()* or after a *body()*
call returns. The script will fail if the headers are modified in any
other situation.

Returns a
`header object <config_http_filters_lua_header_wrapper>`{.interpreted-text
role="ref"}.

### body()

``` {.lua}
local body = handle:body()
```

Returns the stream\'s body. This call will cause Envoy to suspend
execution of the script until the entire body has been received in a
buffer. Note that all buffering must adhere to the flow-control policies
in place. Envoy will not buffer more data than is allowed by the
connection manager.

Returns a
`buffer object <config_http_filters_lua_buffer_wrapper>`{.interpreted-text
role="ref"}.

### bodyChunks()

``` {.lua}
local iterator = handle:bodyChunks()
```

Returns an iterator that can be used to iterate through all received
body chunks as they arrive. Envoy will suspend executing the script in
between chunks, but *will not buffer* them. This can be used by a script
to inspect data as it is streaming by.

``` {.lua}
for chunk in request_handle:bodyChunks() do
  request_handle:log(0, chunk:length())
end
```

Each chunk the iterator returns is a
`buffer object <config_http_filters_lua_buffer_wrapper>`{.interpreted-text
role="ref"}.

### trailers()

``` {.lua}
local trailers = handle:trailers()
```

Returns the stream\'s trailers. May return nil if there are no trailers.
The trailers may be modified before they are sent to the next filter.

Returns a
`header object <config_http_filters_lua_header_wrapper>`{.interpreted-text
role="ref"}.

### log\*()

``` {.lua}
handle:logTrace(message)
handle:logDebug(message)
handle:logInfo(message)
handle:logWarn(message)
handle:logErr(message)
handle:logCritical(message)
```

Logs a message using Envoy\'s application logging. *message* is a string
to log.

### httpCall()

``` {.lua}
local headers, body = handle:httpCall(cluster, headers, body, timeout, asynchronous)
```

Makes an HTTP call to an upstream host. *cluster* is a string which maps
to a configured cluster manager cluster. *headers* is a table of
key/value pairs to send (the value can be a string or table of strings).
Note that the *:method*, *:path*, and *:authority* headers must be set.
*body* is an optional string of body data to send. *timeout* is an
integer that specifies the call timeout in milliseconds.

*asynchronous* is a boolean flag. If asynchronous is set to true, Envoy
will make the HTTP request and continue, regardless of the response
success or failure. If this is set to false, or not set, Envoy will
suspend executing the script until the call completes or has an error.

Returns *headers* which is a table of response headers. Returns *body*
which is the string response body. May be nil if there is no body.

### respond()

``` {.lua}
handle:respond(headers, body)
```

Respond immediately and do not continue further filter iteration. This
call is *only valid in the request flow*. Additionally, a response is
only possible if the request headers have not yet been passed to
subsequent filters. Meaning, the following Lua code is invalid:

``` {.lua}
function envoy_on_request(request_handle)
  for chunk in request_handle:bodyChunks() do
    request_handle:respond(
      {[":status"] = "100"},
      "nope")
  end
end
```

*headers* is a table of key/value pairs to send (the value can be a
string or table of strings). Note that the *:status* header must be set.
*body* is a string and supplies the optional response body. May be nil.

### metadata()

``` {.lua}
local metadata = handle:metadata()
```

Returns the current route entry metadata. Note that the metadata should
be specified under the filter name i.e. *envoy.filters.http.lua*. Below
is an example of a *metadata* in a
`route entry <envoy_v3_api_msg_config.route.v3.Route>`{.interpreted-text
role="ref"}.

``` {.yaml}
metadata:
  filter_metadata:
    envoy.filters.http.lua:
      foo: bar
      baz:
        - bad
        - baz
```

Returns a
`metadata object <config_http_filters_lua_metadata_wrapper>`{.interpreted-text
role="ref"}.

### streamInfo()

``` {.lua}
local streamInfo = handle:streamInfo()
```

Returns
`information <include/envoy/stream_info/stream_info.h>`{.interpreted-text
role="repo"} related to the current request.

Returns a
`stream info object <config_http_filters_lua_stream_info_wrapper>`{.interpreted-text
role="ref"}.

### connection()

``` {.lua}
local connection = handle:connection()
```

Returns the current request\'s underlying
`connection <include/envoy/network/connection.h>`{.interpreted-text
role="repo"}.

Returns a
`connection object <config_http_filters_lua_connection_wrapper>`{.interpreted-text
role="ref"}.

### importPublicKey()

``` {.lua}
local pubkey = handle:importPublicKey(keyder, keyderLength)
```

Returns public key which is used by
`verifySignature <verify_signature>`{.interpreted-text role="ref"} to
verify digital signature.

### verifySignature() {#verify_signature}

``` {.lua}
local ok, error = verifySignature(hashFunction, pubkey, signature, signatureLength, data, dataLength)
```

Verify signature using provided parameters. *hashFunction* is the
variable for the hash function which be used for verifying signature.
*SHA1*, *SHA224*, *SHA256*, *SHA384* and *SHA512* are supported.
*pubkey* is the public key. *signature* is the signature to be verified.
*signatureLength* is the length of the signature. *data* is the content
which will be hashed. *dataLength* is the length of data.

The function returns a pair. If the first element is *true*, the second
element will be empty which means signature is verified; otherwise, the
second element will store the error message.

### base64Escape() {#config_http_filters_lua_stream_handle_api_base64_escape}

``` {.lua}
local base64_encoded = handle:base64Escape("input string")
```

Encodes the input string as base64. This can be useful for escaping
binary data.

Header object API {#config_http_filters_lua_header_wrapper}
-----------------

### add()

``` {.lua}
headers:add(key, value)
```

Adds a header. *key* is a string that supplies the header key. *value*
is a string that supplies the header value.

### get()

``` {.lua}
headers:get(key)
```

Gets a header. *key* is a string that supplies the header key. Returns a
string that is the header value or nil if there is no such header.

------------------------------------------------------------------------

``` {.lua}
for key, value in pairs(headers) do
end
```

Iterates through every header. *key* is a string that supplies the
header key. *value* is a string that supplies the header value.

::: {.attention}
::: {.title}
Attention
:::

In the current implementation, headers cannot be modified during
iteration. Additionally, if it is necessary to modify headers after an
iteration, the iteration must first be completed. This means that
[break]{.title-ref} or any other way to exit the loop early must not be
used. This may be more flexible in the future.
:::

### remove()

``` {.lua}
headers:remove(key)
```

Removes a header. *key* supplies the header key to remove.

### replace()

``` {.lua}
headers:replace(key, value)
```

Replaces a header. *key* is a string that supplies the header key.
*value* is a string that supplies the header value. If the header does
not exist, it is added as per the *add()* function.

Buffer API {#config_http_filters_lua_buffer_wrapper}
----------

### length()

``` {.lua}
local size = buffer:length()
```

Gets the size of the buffer in bytes. Returns an integer.

### getBytes()

``` {.lua}
buffer:getBytes(index, length)
```

Get bytes from the buffer. By default Envoy will not copy all buffer
bytes to Lua. This will cause a buffer segment to be copied. *index* is
an integer and supplies the buffer start index to copy. *length* is an
integer and supplies the buffer length to copy. *index* + *length* must
be less than the buffer length.

### setBytes() {#config_http_filters_lua_buffer_wrapper_api_set_bytes}

``` {.lua}
buffer:setBytes(string)
```

Set the content of wrapped buffer with the input string.

Metadata object API {#config_http_filters_lua_metadata_wrapper}
-------------------

### get()

``` {.lua}
metadata:get(key)
```

Gets a metadata. *key* is a string that supplies the metadata key.
Returns the corresponding value of the given metadata key. The type of
the value can be: *nil*, *boolean*, *number*, *string* and *table*.

------------------------------------------------------------------------

``` {.lua}
for key, value in pairs(metadata) do
end
```

Iterates through every *metadata* entry. *key* is a string that supplies
a *metadata* key. *value* is a *metadata* entry value.

Stream info object API {#config_http_filters_lua_stream_info_wrapper}
----------------------

### protocol()

``` {.lua}
streamInfo:protocol()
```

Returns the string representation of
`HTTP protocol <include/envoy/http/protocol.h>`{.interpreted-text
role="repo"} used by the current request. The possible values are:
*HTTP/1.0*, *HTTP/1.1*, and *HTTP/2*.

### dynamicMetadata()

``` {.lua}
streamInfo:dynamicMetadata()
```

Returns a
`dynamic metadata object <config_http_filters_lua_stream_info_dynamic_metadata_wrapper>`{.interpreted-text
role="ref"}.

### downstreamSslConnection()

``` {.lua}
streamInfo:downstreamSslConnection()
```

Returns `information <include/envoy/ssl/connection.h>`{.interpreted-text
role="repo"} related to the current SSL connection.

Returns a downstream
`SSL connection info object <config_http_filters_lua_ssl_socket_info>`{.interpreted-text
role="ref"}.

Dynamic metadata object API {#config_http_filters_lua_stream_info_dynamic_metadata_wrapper}
---------------------------

### get()

``` {.lua}
dynamicMetadata:get(filterName)

-- to get a value from a returned table.
dynamicMetadata:get(filterName)[key]
```

Gets an entry in dynamic metadata struct. *filterName* is a string that
supplies the filter name, e.g. *envoy.lb*. Returns the corresponding
*table* of a given *filterName*.

### set()

``` {.lua}
dynamicMetadata:set(filterName, key, value)
```

Sets key-value pair of a *filterName*\'s metadata. *filterName* is a key
specifying the target filter name, e.g. *envoy.lb*. The type of *key* is
*string*. The type of *value* is any Lua type that can be mapped to a
metadata value: *table*, *numeric*, *boolean*, *string* or *nil*. When
using a *table* as an argument, its keys can only be *string* or
*numeric*.

``` {.lua}
function envoy_on_request(request_handle)
  local headers = request_handle:headers()
  request_handle:streamInfo():dynamicMetadata():set("envoy.filters.http.lua", "request.info", {
    auth: headers:get("authorization"),
    token: headers:get("x-request-token"),
  })
end

function envoy_on_response(response_handle)
  local meta = response_handle:streamInfo():dynamicMetadata():get("envoy.filters.http.lua")["request.info"]
  response_handle:logInfo("Auth: "..meta.auth..", token: "..meta.token)
end
```

------------------------------------------------------------------------

``` {.lua}
for key, value in pairs(dynamicMetadata) do
end
```

Iterates through every *dynamicMetadata* entry. *key* is a string that
supplies a *dynamicMetadata* key. *value* is a *dynamicMetadata* entry
value.

Connection object API {#config_http_filters_lua_connection_wrapper}
---------------------

### ssl()

``` {.lua}
if connection:ssl() == nil then
  print("plain")
else
  print("secure")
end
```

Returns
`SSL connection <include/envoy/ssl/connection.h>`{.interpreted-text
role="repo"} object when the connection is secured and *nil* when it is
not.

Returns an
`SSL connection info object <config_http_filters_lua_ssl_socket_info>`{.interpreted-text
role="ref"}.

SSL connection object API {#config_http_filters_lua_ssl_socket_info}
-------------------------

### peerCertificatePresented()

``` {.lua}
if downstreamSslConnection:peerCertificatePresented() then
  print("peer certificate is presented")
end
```

Returns a bool representing whether the peer certificate is presented.

### peerCertificateValidated()

``` {.lua}
if downstreamSslConnection:peerCertificateVaidated() then
  print("peer certificate is valiedated")
end
```

Returns bool whether the peer certificate was validated.

### uriSanLocalCertificate()

``` {.lua}
-- For example, uriSanLocalCertificate contains {"san1", "san2"}
local certs = downstreamSslConnection:uriSanLocalCertificate()

-- The following prints san1,san2
handle:logTrace(table.concat(certs, ","))
```

Returns the URIs (as a table) in the SAN field of the local certificate.
Returns an empty table if there is no local certificate, or no SAN
field, or no URI SAN entries.

### sha256PeerCertificateDigest()

``` {.lua}
downstreamSslConnection:sha256PeerCertificateDigest()
```

Returns the SHA256 digest of the peer certificate. Returns `""` if there
is no peer certificate which can happen in TLS (non-mTLS) connections.

### serialNumberPeerCertificate()

``` {.lua}
downstreamSslConnection:serialNumberPeerCertificate()
```

Returns the serial number field of the peer certificate. Returns `""` if
there is no peer certificate, or no serial number.

### issuerPeerCertificate()

``` {.lua}
downstreamSslConnection:issuerPeerCertificate()
```

Returns the issuer field of the peer certificate in RFC 2253 format.
Returns `""` if there is no peer certificate, or no issuer.

### subjectPeerCertificate()

``` {.lua}
downstreamSslConnection:subjectPeerCertificate()
```

Return the subject field of the peer certificate in RFC 2253 format.
Returns `""` if there is no peer certificate, or no subject.

### uriSanPeerCertificate()

``` {.lua}
downstreamSslConnection:uriSanPeerCertificate()
```

Returns the URIs (as a table) in the SAN field of the peer certificate.
Returns an empty table if there is no peer certificate, or no SAN field,
or no URI SAN entries.

### subjectLocalCertificate()

``` {.lua}
downstreamSslConnection:subjectLocalCertificate()
```

Returns the subject field of the local certificate in RFC 2253 format.
Returns `""` if there is no local certificate, or no subject.

### urlEncodedPemEncodedPeerCertificate()

``` {.lua}
downstreamSslConnection:urlEncodedPemEncodedPeerCertificate()
```

Returns the URL-encoded PEM-encoded representation of the peer
certificate. Returns `""` if there is no peer certificate or encoding
fails.

### urlEncodedPemEncodedPeerCertificateChain()

``` {.lua}
downstreamSslConnection:urlEncodedPemEncodedPeerCertificateChain()
```

Returnns the URL-encoded PEM-encoded representation of the full peer
certificate chain including the leaf certificate. Returns `""` if there
is no peer certificate or encoding fails.

### dnsSansPeerCertificate()

``` {.lua}
downstreamSslConnection:dnsSansPeerCertificate()
```

Returns the DNS entries (as a table) in the SAN field of the peer
certificate. Returns an empty table if there is no peer certificate, or
no SAN field, or no DNS SAN entries.

### dnsSansLocalCertificate()

``` {.lua}
downstreamSslConnection:dnsSansLocalCertificate()
```

Returns the DNS entries (as a table) in the SAN field of the local
certificate. Returns an empty table if there is no local certificate, or
no SAN field, or no DNS SAN entries.

### validFromPeerCertificate()

``` {.lua}
downstreamSslConnection:validFromPeerCertificate()
```

Returns the time (timestamp-since-epoch in seconds) that the peer
certificate was issued and should be considered valid from. Returns `0`
if there is no peer certificate.

In Lua, we usually use `os.time(os.date("!*t"))` to get current
timestamp-since-epoch in seconds.

### expirationPeerCertificate()

``` {.lua}
downstreamSslConnection:validFromPeerCertificate()
```

Returns the time (timestamp-since-epoch in seconds) that the peer
certificate expires and should not be considered valid after. Returns
`0` if there is no peer certificate.

In Lua, we usually use `os.time(os.date("!*t"))` to get current
timestamp-since-epoch in seconds.

### sessionId()

``` {.lua}
downstreamSslConnection:sessionId()
```

Returns the hex-encoded TLS session ID as defined in RFC 5246.

### ciphersuiteId()

``` {.lua}
downstreamSslConnection:ciphersuiteId()
```

Returns the standard ID (hex-encoded) for the ciphers used in the
established TLS connection. Returns `"0xffff"` if there is no current
negotiated ciphersuite.

### ciphersuiteString()

``` {.lua}
downstreamSslConnection:ciphersuiteString()
```

Returns the OpenSSL name for the set of ciphers used in the established
TLS connection. Returns `""` if there is no current negotiated
ciphersuite.

### tlsVersion()

``` {.lua}
downstreamSslConnection:urlEncodedPemEncodedPeerCertificateChain()
```

Returns the TLS version (e.g., TLSv1.2, TLSv1.3) used in the established
TLS connection.
