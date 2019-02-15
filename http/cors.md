# Cross-origin resource sharing (CORS)

[Wiki](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)
[MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

CORS is used to manage resource usage across domains to prevent one site from making unwanted data requests to another. This is particularly valuable for sites that use request credentials (e.g. cookies) to authenticate requests.

CORS is managed by the browser, not the client.

## Request Methods

There are two types of CORS requests: "safe" and "unsafe".

Safe requests (`GET`, `HEAD` and some `POST` requests) can be made directly. These requests are considered safe because they do not have an affect on the server.

Requests that might result in changes on the server must be "preflighted" with an `OPTIONS` request to validate that the request is allowed. If the request is allowed, then the browser will send the actual request to the server.

## Headers

Validation information for a cross-site request is passed between the browser and the server using header fields.

### Request Header Fields

#### `Origin`

The origin of the CORS request

#### `Access-Control-Request-Method`

A preflight header that specifies the (non-safe) method of the desired request.

#### `Access-Control-Request-Headers`

A preflight header that is a comma separated list of non-safe header fields that will be sent with the request.

### Response Header Fields

#### `Access-Control-Allow-Origin`

If a server wants to make a resource accessible to other origins, it can set a `Access-Control-Allow-Origin` header. The header can either be a wildcard (`*`) to allow any other origin to access the resource or be a list of one or more allowed origins.

```bash
# request
Origin: http://www.example.com

# response
# wildcard for any origin
Access-Control-Allow-Origin: *
# or specify a specific origin
Access-Control-Allow-Origin: http://www.example.com
```

**Note:** A server can support multiple origins. It would compare the requests's origin to its valid origins and echo it in the `Access-Control-Allow-Origin` header.

#### `Access-Control-Allow-Credentials`

Cross-site requests made using `fetch()` or `XMLHTTPRequest` don't send credentials by default. They can be configured to, but if the response doesn't have an `Access-Control-Allow-Credentials` header set to `true`, the response will be rejected.

Credentialed requests must have an origin-specific `Access-Control-Allow-Origin` header, wildcard headers will be rejected.

#### `Access-Control-Allow-Methods`

A header field for the response to a preflight `OPTIONS` request. This is a comma separated list of acceptable HTTP request methods.

#### `Access-Control-Allow-Headers`

A header field for the response to a preflight `OPTIONS` request. This is a comma separated list of acceptable request header fields.
