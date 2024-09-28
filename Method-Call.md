---
title: "Method Call"
---

Call and Reply are objects with properties, they are conceptually similar to HTTP headers. The `parameters` are the payload. Possible local vendor-specific extensions must be nested in an object named after the vendor in reverse-domain notation.

## Call
```nim
(
  method: string,
  parameters: ?object,
  oneway: ?bool,
  more: ?bool,
  upgrade: ?bool
)
```

|Keyword   |Description                                     |
|----------|------------------------------------------------|
|method    |fully-qualified method name _interface_._method_|
|parameters|input parameters                                |
|oneway    |instructs the server to suppress its reply. The server must adhere to the instruction, to allow clients to associate the next reply to the next call issued without `oneway`|
|more      |requests possible multiple replies to the same call. The server's `more` support is optional, if it does not reply with `continues`, the call will behave like a simple method call and the connection is free to be used for the next method call. If the server replies with `continues`, the connection will be busy until the server sends the final message to the call by not adding `continues`. A client which is no longer interested in the replies from the server simply closes the connection|
|upgrade   |requests the connection to be taken over by a custom protocol/payload. After the server has replied with a single message confirming the upgrade, the connection is no longer controlled by varlink|

## Reply
```nim
(
  parameters: ?object,
  continues: ?bool,
  error: ?string
)
```

|Keyword   |Description                                    |
|----------|-----------------------------------------------|
|parameters|the output parameters of the reply or the error|
|continues |instructs the client to expect multiple replies. It was requested with `more`|
|error     |the fully-qualified reverse-domain error name and indicates that the method call has failed|
