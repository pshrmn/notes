# Cookies

[mdn](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

Cookies are data.

A server can set cookies and send them to the client through header fields on a response.

A client can interact with cookies and update their values.

When the client sends another request to the server, the cookies will be attached to the request.

## `Document.cookie`

On the client, the cookie string can be accessed using through the Document using `document.cookie`. This can be used for both reading and writing.

## Header Fields

### `Set-Cookie`

A response header field specifying the key-value pair for a cookie. Multiple `Set-Cookie` headers are used for multiple pairs.

```bash
Set-Cookie: street=sesame
Set-Cookie: color=blue
```

#### Expiration

A cookie without a specified expiration will theoretically expire when a session ends, although the browser may persist them.

An `Expires` date can be attached to a cookie to specify when it should expire.

```bash
Set-Cookie: auth=j90faw; Expires=<some date>;
```

#### Secure

Secure cookies only get sent with HTTPS requests.

```bash
Set-Cookie: auth=afj30; Secure;
```

#### HTTP Only

HTTP only cookies cannot be read using `Document.cookie`
