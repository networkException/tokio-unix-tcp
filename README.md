# tokio-unix-tcp

This crate wraps the `tokio` types for Unix and TCP Listeners, Socket Addresses and Streams in a generic
enum each, with helper functions existing on both variants passed through.

On non Unix systems, all Unix specific behavior is compiled to no ops.

## Types

### Listener

Either a `tokio::net::TcpListener` or `tokio::net::UnixListener`. This wrapper allows binding to either
a path or IP address and port.

Binding returns a `Listener` instance that can be used to await new incoming connections and accept them.

Accepting a connection returns a `Socket` instance with both a `local_addr` and `peer_addr`.
The `local_peer` will be the `SocketAddr` the `Listener` is bound to and `peer_addr` will be the
remote IP address and port for a TCP socket and an unnamed unix socket address
(`UnixSocketAddr::AbstractOrUnnamed`) for a Unix socket.

Use the `Listener::bind_and_prepare_unix` function to remove an existing file at the bind path when using
Unix sockets. This function also allows adjusting the mode of the socket (defaults to `0o222`).

### UnixSocketAddr

A more developer friendly version of `tokio::net::unix::SocketAddr` for the purposes of this crate. Tokio
current does not support abstract Unix sockets, the underlying `mio::net::SocketAddr` does however.

This leaves `tokio::net::unix::SocketAddr` in an awkward state where calling `is_unnamed` could return `false`
while `as_pathname` also returns `None`. The `UnixSocketAddr` solves this by having a unified variant for
unnamed or abstract sockets.

### SocketAddr

Either a `std::net::SocketAddr` or `UnixSocketAddr`. This type is used as the local or peer address of an
established stream.

Converting to a `NamedSocketAddr` using `to_named_socket_addr` may throw in case the unix socket is
`UnixSocketAddr::AbstractOrUnnamed`, which is not representable as `NamedSocketAddr`. See the documentation
below.

### NamedSocketAddr

Either a `std::net::SocketAddr` or `std::path::PathBuf`. This type is used for creating a socket (connecting) or
creating a listener (binding).

This type differs from `SocketAddr` in that it does not have an unnamed variant for the Unix socket. In case Tokio
starts to support abstract unix socket, the `NamedSocketAddr::Unix` variant will also have to support this instead
of just a `std::path::PathBuf`.

Converting to a `SocketAddr` using `to_socket_addr` always succeeds.

### Stream

Either a `tokio::net::TcpStream` or `tokio::net::UnixStream`. This wrapper allows opening a new connection to either
a path or IP address and port.

When connecting succeeds it returns a `Socket` instance with both a `local_addr` and `peer_addr`.
The `local_peer` will be the local IP address and port for a TCP socket and an unnamed unix socket
address (`UnixSocketAddr::AbstractOrUnnamed`) for a Unix socket and `peer_addr` will be the remote
`SocketAddr` (so IP address and port or path) of the server.

## Flags and Compile Targets

Enabling the `serde` flag adds serializer and deserializer helpers for `SocketAddr` and `NamedSocketAddr`.

Compiling on non `unix` systems will exclude all unix specific functionality and imports. TCP will still work
perfectly fine.

## Related work

- [`multisock`](https://crates.io/crates/multisock) for unifying `std::net` and `std::os::unix::net` types
- [`async-uninet`](https://crates.io/crates/async-uninet) for unifying async types from `async_std` and types from `std`

As far as I can tell both of these don't handle the nuances of abstract and unnamed unix sockets very well, which
this create also aims to fix.

## License

This crate is permissively licensed under either the [BSD 2-Clause "Simplified" License](https://spdx.org/licenses/BSD-2-Clause)
or [MIT License](https://spdx.org/licenses/MIT).
