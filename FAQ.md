---
title: "FAQ"
toc: true
---

## What does varlink provide?
Everybody can read it. Varlink uses a simple human and machine readable text file to describe an
interface with all its data types, method calls and errors. The text file, in its simplicity,
does not require any specific knowledge to join a discussion about design, development, and maintenance of an API.

It documents itself, in the sources and on the wire. Varlink interfaces describe themselves readable to
humans and reliably consumable by machines. Verlink services are introspectable at runtime. Varlink interfaces
provide their documentation along with the interface description.

Varlink uses plain text messages, has no magic numbers, no unnamed values, is easily debuggable with
things like [strace](https://en.wikipedia.org/wiki/Strace), [netcat](https://en.wikipedia.org/wiki/Netcat), [jq](https://stedolan.github.io/jq/).

Varlink uses direct connections, has no central message handling component, it is easily debugged, secured,
isolated, tested.

## Which transports can be used to speak varlink?

Varlink can be implemented on any connection-oriented transport.The most common example are streaming
sockets, but it can also be implemented on kernel special files like device nodes or encapsulated in
other protocols, like HTTP, GSS-API or SASL.

## How can I monitor changes or subscribe to signals?
Varlink connections can be turned into monitor connections, they are soft-negotiated between the
client and the service. The client asks the server to possibly reply with more than one reply, the
server replies and indicates that it might have more replies and the client should wait for them.

To avoid race conditions between the reading of the current state and subscribing to updates, a
typical varlink monitor method call first returns the current state and then sends changes in
subsequent replies to the same request on the same connection.

## Why are there no sequence numbers in calls and replies?
Varlink operates on connections, all messages are strictly in order on the same connection and always
received in the order they are sent. Connections support message pipelining, but not multiplexing.
Complex interfaces can use multiple connections at the same time.

## Can the varlink headers be extended?
The varlink message header can be extended by vendors if needed. Added keys should use the
reverse-domain notation including the vendor's name. The concept is similar to the X-prefix used for
HTTP headers, but neither they nor naked keywords should be added by vendors, they may clash with
other vendors or future varlink protocol specifications.

## Why are records separated by a NUL character and do not specify a length?
The simplest possible transport of varlink messages is a raw streaming socket, which forwards JSON
records separated by a NUL character. Conceptually the NUL character belongs to the transport not to
the varlink message. If varlink messages are encapsulated in HTTP, HTTP will specify the record length
 and not use a NUL byte.

## Why are there no unsigned integers, and why are there no integer sizes specified?
Like JSON, varlink is intentionally underspecified in this regard. The implementation should use
integers large enough to carry the signed integer value, and return an error if it cannot handle the
received number.

## The varlink type system specifies features which do not directly map to JSON, how are they serialized?
Enums are serialized as a JSON name/value pair, with the enum value as a string. Maps are serialized
as JSON objects, every map element is represented as a name/value pair.

## Can I transmit file descriptors?
Varlink does intentionally not support features or side-effects specific to transports. Encapsulating
varlink messages in other protocols, passing them over proxies is a requirement of varlink.

## How can I reference a file?
Varlink should be remotable, therefore you should not point a remote service to a file, which is local
 to the client. Just send the content and an identifier/name of the file.

## How can I transfer larger amounts or foreign data?
You can either send the data as a string, or use the "upgraded" connection feature, where you opt-out
of the varlink protocol after calling a method with the ```upgrade``` header field set to ```true```.
After sending this message the connection is yours for sending whatever you want. To speak the varlink
protocol to the service again, you need to open a new connection. Upgraded varlink connections are
similar to the websocket concept.

## When is a ```oneway``` request useful?
Setting ```oneway``` in a method call means, that the service will not send a reply. This might be
useful when many messages are sent as part of one transaction, and only one single reply is needed to
confirm the transaction. It might also be useful for non-critical data like a debugging interface, to
minimize round-trip handling.

The most important reason to add this feature right from the start is, that it could not be added to
the protocol at a later time, because all implementations need to support it right away to correctly
handle the strict message ordering requirement.

## Why are there no error numbers or human readable messages in errors?
Varlink errors are interface-specific and identified by a string. If required, Interfaces could add a
```message``` field to the error parameters which contain human readable text. The primary focus of
varlink errors is machine consumption, in most cases a carefully chosen descriptive CamelCase error
sufficiently describes the error to humans.

## How do I find the service which implements a local interface?
On systems with the ```org.varlink.resolver``` interface reachable on a defined varlink URI, the
client can call the ```method org.varlink.resolver.Resolve(interface: string) -> (address: string)```.

## How are varlink services accessed from a remote machine?
The varlink command line tool, as well as some varlink language bindings offer the possibility to
connect to a service via a ```bridge```.
CLI:
```bash
varlink --bridge "…"
```

* python:
[varlink.Client.new_with_bridge()](https://varlink.org/python/#varlink.Client.new_with_bridge)
* rust:
[Connection::with_bridge()](https://docs.rs/varlink/4.0.0/varlink/struct.Connection.html#method.with_bridge)

The bridge command normally is ```ssh <host> varlink bridge```, calling ```varlink bridge``` on the
remote host. The ```varlink bridge``` command parses all method calls on stdin, queries the
```org.varlink.resolver``` on the remote system for the interface address, connects to this address
and forwards all method calls matching the interface name to the local service on the remote machine.
It reads the replies and passes them back to stdout. Instead of ```ssh``` any other connection command
 could be used. Instead of ```varlink bridge``` any other utility, which offers an equal
 functionality, could be used.

## How can I get the version number of an interface?
Varlink interfaces do not have a version number, they only have a feature set described in detail by
the interface definition, which is part of the wire protocol.

If your interface has users, you should not break the interface, only extend it, never remove or
incompatibly change things which might be already in use. Clients can fully introspect a service and
figure out which features the interface supports; this is more expressive than any simple numbering
scheme. If you break the API or remove stuff, just release it with a new name, commonly done by adding
 a number to the interface name, like ```org.example.interface2```.

## How do I extend an interface in a backwards compatible way?
Varlink does not use positional parameters or fixed-size objects in its interface definition or on the
 wire, all parameters are identified by their name and can be extended later.

Things like this are not needed:
```C
typedef struct {
       /* … */
       /*
        * Reserved space to allow possible future extensions without
        * breaking the ABI. You should not touch these, because the
        * names of these variables may change.
        */
       uint32_t reserved_int1;
       uint32_t reserved_int2;
       uint32_t reserved_int3;
       uint32_t reserved_int4;
} my_service_struct_foo;
```

Extending existing structures should be done via optional fields (nullable type, maybe). The result of
the methods, when passed parameters without the optional fields should be the same as in older
versions. Method parameters can be extended the same way. The expected behavior for omitted
fields/parameters should be documented. Removing fields, types, errors or methods are not backward
compatible and should be avoided.

## How does socket activation work?
Varlink language bindings commonly implement the systemd socket activation protocol. The environment
of the started service will be augmented with the variables LISTEN_FDS, LISTEN_NAMES, LISTEN_PID. See
details at: https://www.freedesktop.org/software/systemd/man/sd_listen_fds.html

## How do I implement exit-on-idle for my service?
Services can be started on-demand with socket activation. When there is no background task running and
no clients connected to the service, the service can decide to close the listening socket and exit. A
new connection from a client will activate the service again. Because varlink is strictly
point-to-point, there is no buffering involved and the service activator will queue all incoming
messages, exit-on-idle can be implemented in a race-free manner.

## How can I restrict the access to my service?
The most commonly used method is to rely on the permissions of a UNIX socket in the file system.
Services which need a more fine-grained access control can check the connection credentials of a unix
socket and decide per call if they want to act on behalf of the client.
If varlink would be wrapped in GSS-API, e.g. kerberos tickets could be used for more fine grained
access control.
