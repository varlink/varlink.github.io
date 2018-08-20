---
title: "FAQ"
---

## What problems does varlink solve?

Creating an API and keeping compatibility is hard.

If you want to support different programming languages, you might want to write a C-library and create
bindings for the library for the other languages.

Remember structs like this?

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

The opposite of an ABI often used is a command line utility with a man page and human readable output.
To make the output machine readable again, one has to go through bash, sed, awk, perl and whatnot. In
the end, one hopes, that the output format never changes unnoticed, not to speak of i18n and a
forgotten "LANG=C".

```bash
# Random stackoverflow answers to get only the IP from ifconfig/ip
$ ip addr show eth0 | awk '$1 == "inet" {gsub(/\/.*$/, "", $2); print $2}'
$ ifconfig eth0 | awk '/inet addr/ {gsub("addr:", "", $2); print $2}'
$ ifconfig eth0 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}'
$ ifconfig | grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" | grep -v 127.0.0.1 | awk '{ print $2 }' | cut -f2 -d:
```

Another approach is to use a defined protocol to speak to a service, which implements the API.

There are of course plenty of protocols to choose from, like D-BUS, Protocol Buffers, gRPC, JSON-RPC, ….

Most of these protocols don't check all of the [ideals](Ideals), which we deemed important:
- introspectable
- extensible
- simple
- discoverable
- human readable
- machine readable
- remotable

## Can I transport file descriptors?

No, because the client can be a __remote__ client.

<!---
## Which transports can be used to speak varlink?
## How are monitors/subscriptions/signals implemented?
## Why are there no sequence numbers in calls and replies?
## Can the header be extended?
## Why are records separated by a NUL character?
## Why does varlink not specify a length?
## Why are there no unsigned integers?
## Why have integers no size specified?
## How are enums serialized?
## How are maps serialized?
## How are files referenced?
## How can I transfer large binary data?
## When is oneway useful?
## Why are there no error numbers or human readable messages in errors?
## How do I find the service which implements a local interface?
## How are varlink services accessed from a remote machine?
## What is the version number of an interface?
## When are maybes/optionals used?
## How do I extend interfaces in a backwards compatible way?
## How does socket activation work?
-->