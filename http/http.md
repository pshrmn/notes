# HTTP (Hypertext Transfer Protocol)

A "request-response" protocol for transferring data between a client and a server.

## Resources

Resources are identified using uniform resource indicators (URI) with a protocol of either `http` or `https`.

## Session

1. Client makes [TCP](#TCP) connection to a port on a server
2. Connection is established, server listens for request
3. Client sends request
4. Server sends [response](https://www.w3.org/Protocols/rfc2616/rfc2616-sec6.html)

### Request

```bash
GET /some-page HTTP/1.1 # request line
Host: www.example.com # request header fields

# message body (optional)
```

#### Methods

* `GET` - Request a resource from the server
* `HEAD` - A `GET`-like response without the body
* `POST` - Request that the server accepts body attached to the request
* `PUT` - Request that attached body is stored at given URI (modify or create)
  * `PATCH` - Request partial modification of the resource
* `DELETE` - Request the specified resource (URI) is deleted
* `TRACE` - Request that the server echos the request
* `OPTIONS` - Request the list of support HTTP methods

### Response

```bash
HTTP/1.1 200 OK # status line
Content-Type: text/html; charset=UTF-8 # response header fields
Content-Encoding: gzip

<html> # message body (optional)
</html>
```

## Transmission Control Protocol (TCP)

[wiki](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)

A server binds itself to a port and listens ("passive open"). An "active open" happens when a client establishes a connection with the server.

### Establishing a connection

1. `SYN` - Client sends random sequence number to the server (`X`).
2. `SYN-ACK` - Server responds with `X + 1` and its own random number `Y`.
3. `ACK` - Client acknowledges by return `Y + 1` (the acknowledgement number). Client stores `X + 1` as the sequence number.

### Terminating a connection

Either the client or server can terminate a connection.

1. `FIN` - Client or server sends `FIN` packet to the other.
2. `ACK` - The other acknowledges by responding with an `ACK`.
3. `FIN` - The other sends its own `FIN` packet.
4. `ACK` - Original `FIN` sender sends `ACK` to the other and waits for some amount of time before closing the connection (to prevent in-transit packets from being used in future connections).
