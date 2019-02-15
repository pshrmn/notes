# Cross-site request forgery (CSRF) Protection

([cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery))

> CSRF exploits the trust that a site has in a user's browser

If a user visits a malicious site, that site attempt to make unwanted requests to another origin. This is known as a cross-site request forgery.

An HTTP request to a server sends cookies. With same-origin CORS requests, cookies are only sent with credentials requests. However, other types of requests work without CORS validation. For example, the malicious site might use an `<img>` tag's `src` element to make a request to another site.

Cross-site request forgery takes advantage of vulnerable resources. This can include form submissions and requests that trigger other actions.

CSRF prevention typically involves setting a unique token per response. When a client submits a request, it should include the token from the server's response.

There are various methods for attaching a token to a request. For form submissions, the token may be included in the serialized form data (e.g. part of a `POST` request's body). Alternatively, a header field (`X-CSRF-Token`) may be passed with the token.
